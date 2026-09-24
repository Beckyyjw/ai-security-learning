# LLM05:2026 — Data and Model Poisoning

## C'est quoi ?
Corrompre les données ou le modèle pour lui faire adopter un comportement
caché, qui s'active seulement dans certaines conditions.
C'est une corruption durable : on ne la corrige pas avec un patch,
il faut revalider les données, réentraîner ou remplacer le modèle.

---

## La différence avec LLM01 et LLM04

| Risque | Quand ça se passe | Ce qui est corrompu |
|---|---|---|
| **LLM01** | À l'inférence (en temps réel) | Le comportement via le prompt |
| **LLM04** | Dans la chaîne (composant tiers) | Un paquet, un modèle, un outil |
| **LLM05** | Dans les données ou le modèle | L'apprentissage lui-même, de façon durable |

---

## Les 5 moments où ça peut se produire

| Moment | Ce qui est ciblé |
|---|---|
| **Pré-entraînement** | Les données d'entraînement générales |
| **Fine-tuning** | Les données d'adaptation à une tâche |
| **Embeddings** | Les vecteurs stockés dans la base RAG |
| **Transfer learning** | Un modèle source compromis transmis à un modèle dérivé |
| **Apprentissage continu** | Des données injectées progressivement dans une boucle automatique |

---

## Le concept clé : le sleeper agent

Une porte dérobée laisse le comportement du modèle intact jusqu'à ce
qu'un déclencheur l'active. Le modèle passe tous les tests, semble
parfaitement normal, et change de comportement uniquement sur ce déclencheur.

Un déclencheur peut être : un mot précis, une phrase, une image particulière,
un contexte spécifique.

---

## Chiffres marquants

- **250 documents empoisonnés** suffisent à compromettre des modèles
  de 600M à 13 milliards de paramètres, quelle que soit la taille du dataset.
- Modifier le **template de chat** peut faire chuter l'exactitude
  de **90% à 15%** quand le déclencheur est présent.
- **L'entraînement à la sécurité ne supprime pas les portes dérobées.**

---

## Point clé pour un agent

La mémoire persistante de l'agent est une cible d'empoisonnement.
Des instructions injectées sur plusieurs sessions peuvent orienter
durablement son comportement, sans qu'aucune session individuelle
ne semble suspecte.

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Tracer l'origine des datasets et modèles** à chaque étape |
| ⭐⭐⭐ | **Traiter les templates de chat, tokenizer configs et adaptateurs comme du code** : signature, vérification d'empreinte |
| ⭐⭐⭐ | **Contrôler les boucles de réentraînement automatique** avec validation humaine |
| ⭐⭐ | **Versionner les données** (DVC) pour pouvoir revenir en arrière |
| ⭐⭐ | **Rechercher activement les déclencheurs** après chaque cycle d'alignement |
| ⭐ | Sandboxing strict pour limiter l'exposition à des sources non vérifiées |

---

## Liens avec les autres risques

- → **LLM04 Supply Chain** : LLM04 = composant tiers compromis, LLM05 = données ou comportement corrompus
- → **LLM01 Prompt Injection** : injection au moment de l'inférence, LLM05 = corruption durable
- → **LLM09 Vector & Embedding Weaknesses** : empoisonnement des embeddings RAG

---

## En une phrase
> L'empoisonnement touche l'apprentissage du modèle, pas un bug ponctuel.
> On ne le corrige pas avec un patch : il faut tout reprendre depuis les données.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM05*
