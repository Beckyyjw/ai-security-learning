# Phase 3 — Les défenses

> Cette phase implémente les contre-mesures qui neutralisent les 3 attaques de la
> Phase 2. **Principe directeur : la sécurité vit dans le code, pas dans le
> modèle.** Le comportement du LLM est non déterministe (cf. Phase 2) ; les défenses,
> elles, sont déterministes et s'appliquent à 100 % des exécutions.

---

## L'architecture : un poste de contrôle

Toutes les défenses passent par une **fonction de contrôle unique**,
`controle_securite(nom, args)`, appelée **avant chaque exécution d'outil** dans la
boucle de l'agent. Elle renvoie `(autorisé: bool, raison: str)`.

```python
for appel in msg["tool_calls"]:
    nom  = appel["function"]["name"]
    args = appel["function"]["arguments"]

    # 🛡️ Contrôle AVANT exécution
    autorise, raison = controle_securite(nom, args)
    if not autorise:
        print(f"[BLOQUÉ : {nom} — {raison}]")
        messages.append({"role": "tool", "content": f"Action refusée : {raison}"})
        continue

    # 🧑 Human-in-the-loop pour les actions sensibles
    if nom in OUTILS_SENSIBLES:
        if input(f"Autoriser {nom} ? (oui/non) : ").strip().lower() != "oui":
            print(f"[REFUSÉ PAR L'HUMAIN : {nom}]")
            messages.append({"role": "tool", "content": "Action refusée par l'humain."})
            continue

    resultat = OUTILS_DISPO[nom](**args)
    messages.append({"role": "tool", "content": str(resultat)})
```

C'est le point unique où toute action est vérifiée : le **choke point** de sécurité.

---

## Défense 1 — Moindre privilège (allowlist d'outils)

**Contre l'attaque 3 (suppression / excessive agency).**

Seuls les outils explicitement autorisés peuvent s'exécuter. `supprimer_fichier`
n'y figure pas → toute demande de suppression est refusée.

```python
OUTILS_AUTORISES = ["lire_fichier", "envoyer_donnees"]

if nom not in OUTILS_AUTORISES:
    return False, f"Outil '{nom}' interdit (pas dans la liste blanche)."
```

**Résultat testé :** `[BLOQUÉ : supprimer_fichier …]` — le fichier survit (vérifié par `dir`). ✅

---

## Défense 2 — Allowlist d'URL

**Contre l'attaque 2 (exfiltration).**

`envoyer_donnees` est légitime, mais ne doit envoyer que vers des destinations
autorisées. Toute autre URL est bloquée.

```python
URLS_AUTORISEES = ["https://api.entreprise-legitime.com"]

if nom == "envoyer_donnees":
    url = args.get("url", "")
    if not any(url.startswith(u) for u in URLS_AUTORISEES):
        return False, f"URL non autorisée : {url}"
```

**Résultat testé :** `[BLOQUÉ : envoyer_donnees — URL non autorisée …]` — même les
variantes de l'URL malveillante (`?token=…`) sont bloquées (vérification par
préfixe). ✅

---

## Défense 3 — Protection des fichiers sensibles

**Défense en profondeur contre le vol du secret.**

Un fichier sensible ne doit pas être lisible pour une tâche de résumé.

```python
FICHIERS_INTERDITS = ["secrets.txt"]

if nom == "lire_fichier":
    chemin = args.get("chemin", "")
    if any(interdit in chemin for interdit in FICHIERS_INTERDITS):
        return False, f"Accès refusé au fichier sensible : {chemin}"
```

**Résultat testé :** `[BLOQUÉ : lire_fichier — Accès refusé au fichier sensible :
secrets.txt]` — le secret n'est même plus **lisible**. Combinée à la Défense 2,
l'exfiltration est bloquée à **deux niveaux**. ✅

---

## Défense 4 — Human-in-the-loop

**Contre toute action sensible, même autorisée.**

Pour les actions à fort impact (envoi, suppression), une **confirmation humaine**
est exigée avant l'exécution. L'humain a le dernier mot.

```python
OUTILS_SENSIBLES = ["envoyer_donnees", "supprimer_fichier"]
```

**Résultat testé :** sur un envoi vers une URL pourtant autorisée, le programme
s'arrête et demande confirmation ; un `non` produit
`[REFUSÉ PAR L'HUMAIN : envoyer_donnees]`. ✅

> **Arbitrage :** on ne met en `OUTILS_SENSIBLES` que les actions vraiment à risque
> (pas la simple lecture), pour ne pas rendre l'agent pénible à l'usage. Sécurité vs
> confort : un vrai compromis d'ingénierie.

---

## Bilan attaques → défenses

| Attaque | Défense(s) | Statut |
|---|---|---|
| 1 — Injection / goal hijacking | Séparation données/instructions (atténuation) + HITL | ⚠️ atténuée |
| 2 — Exfiltration | Allowlist URL (D2) + fichier sensible (D3) | ✅ bloquée |
| 3 — Suppression / excessive agency | Moindre privilège (D1) + HITL (D4) | ✅ bloquée |

> **Note honnête :** l'injection de prompt (attaque 1) ne se « corrige » pas
> totalement — c'est un problème de recherche ouvert. Les défenses la **réduisent**
> et **limitent son impact** (l'agent détourné ne peut plus rien exfiltrer ni
> détruire), mais aucune barrière n'empêche à 100 % un LLM de se laisser influencer
> par un texte. D'où l'importance de la défense en profondeur.

## Principe à retenir

> La sécurité d'un agent ne repose jamais sur le bon comportement du modèle
> (non déterministe, manipulable, capable de mentir). Elle repose sur des
> **contrôles déterministes dans le code**, appliqués à un **point de passage
> unique**, selon le **moindre privilège** et la **défense en profondeur**, avec un
> **humain dans la boucle** pour les actions critiques.
