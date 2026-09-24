# LLM10:2026 — Improper Output Handling

## C'est quoi ?
La sortie du modèle est transmise à d'autres composants
sans validation ni nettoyage.
Comme cette sortie peut être contrôlée par un prompt,
l'attaquant obtient un accès indirect à ces composants.

C'est de la sécurité web classique appliquée aux LLM.

---

## La différence avec LLM07

| Risque | Ce qui pose problème |
|---|---|
| **LLM07 Misinformation** | La sortie est fausse ou trompeuse |
| **LLM10 Improper Output Handling** | La sortie est dangereuse pour le système qui la reçoit |

---

## Les vulnérabilités classiques reproduites

| Sortie non validée | Conséquence |
|---|---|
| Passée à `exec` ou `eval` | Exécution de code à distance (RCE) |
| JavaScript ou Markdown affiché dans le navigateur | XSS |
| Requête SQL sans paramétrage | Injection SQL |
| Chemin de fichier construit depuis la sortie | Path traversal |
| Contenu dans un template email sans échappement | Phishing |
| Code généré compilé et déployé sans revue | Vulnérabilités en production |

---

## Les nouveautés 2026

**Séquences d'échappement ANSI**
Des caractères de contrôle écrits dans un terminal, un log ou un IDE
permettent de tromper l'affichage ou de détourner le presse-papier.

**Images Markdown affichées automatiquement**
L'URL de l'image peut exfiltrer le contenu de la conversation
vers le serveur de l'attaquant via le nom de domaine ou les paramètres de l'URL.

---

## Conditions qui aggravent le risque

- L'agent a des privilèges élevés (RCE possible)
- L'application est vulnérable à l'injection indirecte (LLM01)
- Les extensions tierces ne valident pas leurs entrées
- Pas d'encodage contextuel des sorties
- Pas de monitoring des sorties du modèle

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Traiter le modèle comme un utilisateur non fiable** : zero trust sur ses sorties |
| ⭐⭐⭐ | **Encodage contextuel** : HTML pour le web, SQL escaping pour les bases, JS encoding pour les scripts |
| ⭐⭐⭐ | **Requêtes paramétrées** pour toutes les opérations base de données impliquant une sortie LLM |
| ⭐⭐ | **CSP stricte** pour limiter les XSS depuis du contenu généré |
| ⭐⭐ | **Nettoyer les caractères de contrôle** (ANSI, BEL, OSC) avant tout affichage terminal ou log |
| ⭐⭐ | **Désactiver l'affichage automatique des images Markdown** ou le limiter à une liste d'origines autorisées |
| ⭐ | Logging et monitoring pour détecter les patterns anormaux |

---

## Liens avec les autres risques

- → **LLM01 Prompt Injection** : l'injection fournit la sortie malveillante, LLM10 l'exécute
- → **LLM03 Excessive Agency** : LLM03 = droits trop larges, LLM10 = sortie non validée
- → **LLM07 Misinformation** : LLM07 = info fausse, LLM10 = sortie dangereuse pour le système

---

## En une phrase
> Chaque sortie de ton agent qui alimente un outil ou un système
> doit être validée comme une entrée utilisateur non fiable.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM10*
