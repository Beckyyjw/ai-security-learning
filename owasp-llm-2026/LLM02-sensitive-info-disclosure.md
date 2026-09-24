# LLM02:2026 — Sensitive Information Disclosure

## C'est quoi ?
Le système expose des données confidentielles par un canal non autorisé.
La fuite ne passe pas seulement par la réponse finale : logs, traces de
raisonnement, arguments d'outils et même le temps de réponse peuvent
divulguer des informations sensibles.

> Le problème est souvent **en amont** : des permissions trop larges
> alimentent le RAG avec des données sensibles. Il faut corriger la
> source, pas le modèle.

---

## Les 4 moments où la fuite peut se produire

| Moment | Ce qui fuite | Exemple réel |
|---|---|---|
| **Entraînement** | Données mémorisées par le modèle | GPT-3.5 : 10 000 exemples ressortis pour ~200$ |
| **Inférence** | Contexte en cours (system prompt, RAG, autre session) | Bug Redis ChatGPT 2023 : données de paiement d'un utilisateur vues par un autre |
| **Pipeline** | Outils de monitoring, fine-tuning, données synthétiques | DeepSeek jan. 2025 : 1M+ lignes de logs et clés API exposées |
| **Observation** | Temps de réponse, taille des tokens, scores de similarité | Whisper Leak : sujets de conversation reconstitués à 98% sur trafic chiffré |

---

## Les canaux de fuite (souvent oubliés)

| Canal | Exemple concret |
|---|---|
| Réponse finale | Le modèle répète un email ou un mot de passe |
| Arguments d'outils | L'agent appelle une API avec des données sensibles en paramètre |
| Traces de raisonnement | Le modèle "pense à voix haute" et expose des données dans ses étapes |
| Logs et observabilité | LangSmith, Datadog enregistrent tout par défaut |
| Temps de réponse | Un attaquant déduit si une donnée existe en mesurant le délai |

---

## Les 2 causes structurelles

**1. Partage excessif en amont**
Permissions trop larges → le RAG récupère des données sensibles
exactement comme prévu. Le problème est dans la source, pas le modèle.

**2. La persistance**
Une fois qu'une donnée a influencé les poids ou les embeddings,
elle reste extractible même après suppression de la source.
→ Problème légal : RGPD Article 17 (droit à l'effacement)

---

## Point clé sur les embeddings

Un embedding = un texte converti en liste de nombres.
Une **embedding inversion attack** reconvertit ces nombres en texte original.


**Vec2Text** reconstruit jusqu'à **92% des mots** de textes courts
sans jamais avoir eu accès aux documents originaux.

➡ Une fuite d'embeddings = une fuite de documents.
➡ Couvert en détail par LLM09.

---

## Comment un attaquant récupère les vecteurs

| Méthode | Comment |
|---|---|
| **Fuite directe** | Bucket S3 public, base sans mot de passe, backup exposé |
| **API mal sécurisée** | L'API retourne les scores ou vecteurs bruts |
| **Interrogation répétée** | Des milliers de requêtes pour reconstituer les vecteurs par déduction |
| **Side-channel** | Mesure du temps de réponse et de la taille des réponses |

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Autoriser avant de récupérer** : vérifier les droits dans la requête à l'index, pas après |
| ⭐⭐⭐ | **Jamais de secrets dans le system prompt** |
| ⭐⭐⭐ | **Envoyer au modèle seulement les champs nécessaires** |
| ⭐⭐ | **Nettoyer logs et traces** avant de les stocker dans un outil d'observabilité |
| ⭐⭐ | **Traiter les traces de raisonnement** comme des sorties à filtrer |
| ⭐⭐ | **Chiffrer les bases vectorielles** et les traiter comme les documents sources |
| ⭐ | Ne jamais renvoyer les scores de similarité bruts au client |

---

## Liens avec les autres risques

- → **LLM01 Prompt Injection** : une injection peut forcer le modèle à révéler son contexte
- → **LLM08 Hidden Context Exposure** : fuite du system prompt = cas particulier de LLM02
- → **LLM09 Vector & Embedding Weaknesses** : les embeddings peuvent être inversés

---

## Cas réels

- **ChatGPT Redis bug (2023)** : données de paiement d'un utilisateur exposées à un autre
- **DeepSeek (jan. 2025)** : 1M+ lignes de logs et clés API exposées publiquement
- **Whisper Leak** : sujets de conversation reconstitués à >98% sur trafic chiffré
- **GPT-3.5** : 10 000+ exemples mémorisés ressortis pour ~200$

---

## En une phrase
> Le modèle peut divulguer des données sensibles par sa réponse, ses logs,
> ses traces ou même son temps de réponse. La source du problème est
> souvent en amont, dans les données auxquelles on lui donne accès.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM02*
