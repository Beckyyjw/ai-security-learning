# LLM06:2026 — Unbounded Consumption

## C'est quoi ?
L'application permet une consommation de ressources sans contrôle :
déni de service, factures insoutenables, ou vol du modèle par clonage.

L'idée centrale : **l'asymétrie des coûts.**
L'attaquant déclenche des calculs très coûteux pour un coût dérisoire de son côté.

---

## Les risques principaux

| Risque | Ce qui se passe |
|---|---|
| **Variable-Length Input Flood** | Entrées de tailles variées saturent le système |
| **Denial of Wallet (DoW)** | L'attaquant fait exploser la facture d'un service payé à l'usage |
| **Boucles de raisonnement** | Un prompt court pousse un modèle à raisonner indéfiniment |
| **Boucles d'appels d'outils** | Un outil malveillant pousse l'agent dans des appels récursifs |
| **Contexte qui grossit** | Chaque tour retraite tout l'historique, le coût monte exponentiellement |
| **Extraction du modèle** | Copier un modèle en l'interrogeant massivement |

---

## Le cas du contexte qui grossit (important pour un agent)

Dans une longue session agentique, chaque tour retraite tout l'historique.

Exemple du document :
- Tour 1 : ~0,001$
- Tour 100 : ~0,50$

Aucune requête individuelle ne dépasse les limites.
L'agrégat sur des sessions longues ou nombreuses atteint des centaines de dollars.

---

## Chiffre marquant

Un seul sample d'entraînement malveillant peut briser le comportement
de fin de séquence du modèle et pousser chaque réponse au maximum
de longueur autorisée.

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Limites en tokens et en coût**, pas seulement en nombre de requêtes |
| ⭐⭐⭐ | **Plafonds de dépense bloquants** : arrêtent l'inférence, pas de simples alertes |
| ⭐⭐⭐ | **Coupe-circuits pour agents** : limite d'étapes, de récursion, de temps et de coût par exécution |
| ⭐⭐ | Détection de boucles par hachage d'état |
| ⭐⭐ | Mise à jour des frameworks de serving (vLLM, Ollama, etc.) |
| ⭐ | Dégradation gracieuse sous charge : maintenir une fonctionnalité partielle plutôt qu'une panne totale |

---

## Liens avec les autres risques

- → **LLM03 Excessive Agency** : un agent trop autonome peut déclencher des cascades d'appels coûteux
- → **LLM04 Supply Chain** : un outil MCP malveillant peut forcer des boucles d'appels

---

## En une phrase
> L'attaquant déclenche des calculs très coûteux pour presque rien.
> Les coupe-circuits sont indispensables pour tout agent autonome.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM06*
