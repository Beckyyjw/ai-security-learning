# 🛡️ Agentic Security Lab

> Un laboratoire pratique sur la **sécurité des agents IA** : je construis un agent
> volontairement vulnérable, je l'attaque (injection, exfiltration, excessive
> agency), puis je code les défenses qui neutralisent ces attaques.
>
> **Thèse du projet :** la sécurité d'un agent IA ne réside pas dans le modèle
> (imprévisible et manipulable), mais dans le **code déterministe** qui contrôle ses
> actions.

Projet réalisé par **Becky Joyce ELIBE** — étudiante ingénieure en Réseaux &
Sécurité (EFREI Paris).

---

## 🎯 Le concept

Un **agent IA** ne se contente pas de répondre : il *agit* (lire des fichiers,
envoyer des données…) via des **outils**, en boucle, jusqu'à accomplir sa tâche.
Cette capacité d'action est aussi sa principale surface d'attaque.

Ce lab démontre, preuves à l'appui, trois familles d'attaques puis leurs
contre-mesures — le tout tournant **en local** (Ollama + llama3.1:8b), sans aucune
clé API.

## 🏗️ Architecture

```
Utilisateur ──▶ Agent (LLM) ──▶ [demande d'outil]
                                      │
                             🛡️ contrôle_securite()   ← la sécurité vit ICI
                                      │
                             OUTILS_DISPO[nom](**args) ← exécution réelle (code)
```

Le LLM ne fait que **demander** un outil ; c'est le **code** qui exécute — et donc
qui décide d'autoriser ou de refuser.

## ⚔️ Phase 2 — Les attaques

| # | Attaque | Principe | OWASP LLM |
|---|---|---|---|
| 1 | **Injection indirecte** (goal hijacking) | Des instructions cachées dans un fichier détournent l'agent | LLM01 |
| 2 | **Exfiltration de données** | L'agent est poussé à envoyer un secret vers un serveur attaquant | LLM01/LLM02/LLM06 |
| 3 | **Excessive agency** | Un agent doté d'un pouvoir inutile (suppression) est détourné pour détruire des données | LLM08 |

**Comportement du modèle observé (finding central) :** pour une même faille, le
modèle a réagi de façon **totalement non déterministe** — obéissance, refus,
**mensonge** (affirme une action non faite), **hallucination** (invente un fichier)
et **confabulation** (fabrique de faux identifiants). → *On ne peut jamais se fier
au comportement du modèle comme mécanisme de sécurité.*

## 🛡️ Phase 3 — Les défenses

Toutes centralisées dans une fonction `controle_securite()` appelée avant chaque
exécution d'outil (**point de passage unique**) :

| Défense | Principe | Neutralise |
|---|---|---|
| 1. **Moindre privilège** | Allowlist d'outils | Attaque 3 |
| 2. **Allowlist d'URL** | Envois vers destinations autorisées uniquement | Attaque 2 |
| 3. **Protection des fichiers sensibles** | Lecture des secrets interdite | Vol du secret |
| 4. **Human-in-the-loop** | Confirmation humaine avant action sensible | Actions critiques |

Contrairement au modèle, ces défenses sont **déterministes** : elles s'appliquent à
**100 % des exécutions**, quoi que le LLM décide.

## 📸 Démonstrations



**Attaque 1 — Injection indirecte** (l'agent répond « SYSTÈME COMPROMIS ») :
 ![Attaque 1](docs/captures/attaque-1-resultat.png) 


**Attaque 2 — Exfiltration** (envoi vers l'URL de l'attaquant + log) :
 ![Attaque 2](docs/captures/attaque-2-log.png) 

**Attaque 3 — Suppression** (fichier réellement effacé) :
 ![Attaque 3](docs/captures/attaque-3-suppression.png) 


**Défenses en action** (les `🛡️ BLOQUÉ` et le `🧑 REFUSÉ PAR L'HUMAIN`) :
 ![Défenses](docs/captures/defenses-bloque.png) 


## 🧰 Stack technique

- **Python**
- **Ollama** (exécution locale du modèle, gratuit, offline)
- Modèle : **llama3.1:8b** (tool calling)

## 📁 Structure du dépôt

```
agentic-security-lab/
├── agent.py                  # l'agent (vulnérable → sécurisé)
├── README.md                 # ce fichier
└── docs/
    ├── carnet-apprentissage.md        # le raisonnement, question par question
    ├── attaque-1-injection-indirecte.md
    ├── attaque-2-exfiltration.md
    ├── attaque-3-excessive-agency.md
    ├── phase3-defenses.md
    └── captures/             # captures d'écran des démonstrations
```

## 🚀 Lancer le projet

```bash
# 1. Installer Ollama : https://ollama.com
ollama pull llama3.1:8b

# 2. Dépendance Python
pip install ollama

# 3. Lancer l'agent
python agent.py
```

## 💡 Ce que ce projet démontre

1. Comment fonctionne réellement un agent IA (boucle LLM ↔ outils).
2. Pourquoi les agents sont vulnérables : **le LLM ne distingue pas une donnée
   d'une instruction.**
3. Que le comportement d'un LLM est **non déterministe** — donc inutilisable comme
   garde-fou.
4. Comment sécuriser un agent par le **code** : moindre privilège, allowlists,
   défense en profondeur, human-in-the-loop.

## 📚 Références

- OWASP Top 10 for LLM Applications
- MITRE ATLAS
- *Lethal trifecta* (données sensibles + contenu non fiable + canal de sortie)

---

*Lab d'apprentissage — les fichiers `secrets.txt`, URLs et données sont fictifs et
servent uniquement à la démonstration.*
