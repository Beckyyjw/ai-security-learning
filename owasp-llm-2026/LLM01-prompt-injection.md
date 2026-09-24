# LLM01:2026 — Prompt Injection

## C'est quoi ?
Une entrée malveillante modifie le comportement du modèle d'une façon non prévue
par le développeur. Le LLM ne distingue pas « instruction » et « donnée » :
tout est du texte dans le même flux (context window).

> Pas d'équivalent aux requêtes paramétrées qui ont résolu l'injection SQL.
> On ne peut pas l'empêcher totalement — on limite les dégâts.

---

## Les 2 types

| Type | Comment | Qui le voit |
|---|---|---|
| **Directe** | L'utilisateur écrit l'attaque lui-même | L'utilisateur |
| **Indirecte** | Cachée dans un contenu lu par le modèle (email, page web, ticket, RAG) | Personne |

**Jailbreak** = cas particulier de directe : faire sauter les règles de sécurité du modèle.

---

## Pourquoi c'est pire avec un agent

Un chatbot peut **dire** quelque chose de faux.
Un agent peut **faire** quelque chose de dangereux (shell, email, API, MCP).

Scénario type :
1. Attaquant dépose une instruction piégée dans un ticket ou une issue GitHub
2. L'agent la lit avec les droits élevés de l'utilisateur
3. L'agent exécute lui-même l'action (exfiltration, suppression, envoi)
4. L'attaquant n'a jamais eu besoin d'accéder au système

---

## Techniques d'attaque

| Technique | Principe |
|---|---|
| Texte invisible | Blanc sur blanc, commentaires HTML |
| Unicode invisible | Caractères que l'humain ne voit pas (zero-width, variation selectors) |
| Encodage | Base64, langue rare, emojis |
| Payload splitting | Attaque découpée en morceaux, recombinée par le modèle |
| Injection multimodale | Instructions cachées dans une image ou un audio |
| Empoisonnement mémoire | Instruction écrite en mémoire persistante, active à chaque session |

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Moindre privilège** : l'agent n'a que les droits strictement nécessaires |
| ⭐⭐⭐ | **Secrets dans le code**, jamais dans le prompt |
| ⭐⭐⭐ | **Rule of Two** : pas plus de 2 parmi A (entrées non fiables) / B (données sensibles) / C (actions externes) |
| ⭐⭐ | **Human-in-the-loop** : validation humaine avant toute action irréversible |
| ⭐⭐ | **Policy engine déterministe** : c'est le code qui autorise les actions, pas le LLM |
| ⭐ | Supprimer les caractères Unicode invisibles à l'entrée et à l'affichage |
| ⭐ | Épingler et signer les serveurs MCP |

---

## Rule of Two (rappel)

Ne jamais donner les 3 capacités suivantes en même temps à un agent :
- **A** — lire du contenu non fiable (emails reçus, pages web, tickets)
- **B** — accéder à des données sensibles (base clients, fichiers internes)
- **C** — agir vers l'extérieur (envoyer, modifier, supprimer)

Si les 3 sont nécessaires → **validation humaine obligatoire pour chaque action**.

---

## Liens avec les autres risques

- → **LLM03 Excessive Agency** : l'injection est l'entrée, l'excès d'autonomie amplifie les dégâts
- → **LLM02 Sensitive Info Disclosure** : ce que le modèle peut divulguer une fois injecté
- → **LLM08 Hidden Context Exposure** : si le system prompt fuite, l'attaque directe devient plus précise

---

## Cas réels

- **Microsoft 365 Copilot** : email piégé → exfiltration sans clic utilisateur
- **Supabase MCP (Cursor)** : base de données vidée via un agent trop privilégié
- **Invariant Labs** : exfiltration de dépôts privés via une issue GitHub piégée
- **postmark-mcp** : ~300 organisations avec l'attaquant en copie cachée de leurs emails

---

## En une phrase
> Le LLM ne distingue pas ordre et donnée.
> On ne peut pas l'empêcher d'être trompé, mais on peut l'empêcher de faire des dégâts.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM01*
