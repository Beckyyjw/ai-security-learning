# LLM08:2026 — Hidden Context Exposure

## C'est quoi ?
L'extraction ou la reconstruction du contexte caché que l'utilisateur
n'est pas censé voir : system prompt, instructions du développeur,
politiques récupérées, schémas des outils exposés au modèle.

L'idée clé : **le contexte caché n'est pas un secret.**
Il faut partir du principe qu'il sera découvert.
Le vrai problème n'est pas la fuite en soi, mais ce qu'on y a mis
(des identifiants) ou le fait qu'on compte sur lui pour la sécurité.

---

## L'échelle de gravité

| Niveau | Contenu exposé |
|---|---|
| **Informationnelle** | Pas de secret ni de logique sensible |
| **Moyenne** | Règles internes, critères de filtrage, logique métier |
| **Haute** | Identifiants dans le contexte, sécurité basée sur sa confidentialité |
| **Critique** | Fuite menant à exécution de code, exfiltration massive ou élévation de privilèges |

---

## Ce qui peut être exposé

| Élément | Risque si exposé |
|---|---|
| **Identifiants et clés API** | L'attaquant les utilise directement |
| **Schémas des outils** | L'attaquant cible précisément les outils disponibles |
| **Règles de filtrage** | L'attaquant conçoit des prompts qui contournent les filtres |
| **Rôles et permissions** | L'attaquant cherche une élévation de privilèges |
| **Logique de formatage** | L'attaquant génère des sorties conformes mais malveillantes |

---

## Pourquoi ça amplifie les autres risques

- Règles révélées → injection plus précise **(LLM01)**
- Identifiants exposés → fuite de données **(LLM02)**
- Schémas d'outils révélés → cibles pour l'excès d'autonomie **(LLM03)**
- Logique de formatage révélée → sorties malveillantes conformes **(LLM10)**

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Ne rien mettre de sensible dans le contexte** : ni identifiants, ni secrets, ni logique de sécurité |
| ⭐⭐⭐ | **Partir du principe que le contexte sera découvert** et concevoir en conséquence |
| ⭐⭐⭐ | **Appliquer les comportements critiques avec des mécanismes déterministes** hors du modèle |
| ⭐⭐ | Gérer les autorisations indépendamment du LLM |
| ⭐ | Ne pas compter sur la confidentialité du prompt comme contrôle de sécurité |

---

## Liens avec les autres risques

- → **LLM01 Prompt Injection** : des règles révélées facilitent une injection ciblée
- → **LLM02 Sensitive Info Disclosure** : identifiants dans le contexte = fuite de données
- → **LLM03 Excessive Agency** : schémas d'outils révélés = cibles pour l'attaquant

---

## En une phrase
> Le system prompt n'est pas un coffre-fort.
> Ne mets jamais dedans ce que tu ne voudrais pas voir affiché en clair.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM08*
