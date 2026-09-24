# LLM09:2026 — Vector and Embedding Weaknesses

## C'est quoi ?
Des attaques qui exploitent la géométrie de l'espace des embeddings
et la recherche par similarité.
Ça concerne le RAG, la mémoire vectorielle des agents,
les caches sémantiques et la déduplication.

Contrairement à LLM01, ces attaques fonctionnent souvent
**sans aucune instruction malveillante** dans le contenu.

---

## Le moyen mnémotechnique

| Action | Ce que ça fait |
|---|---|
| **Empoisonnement** | Rend le système faux : du contenu est récupéré à la place du bon |
| **Inversion** | Fait fuiter les documents : on reconstruit le texte depuis les vecteurs |
| **Brouillage** | Rend le système muet : un document bloqueur empêche toute réponse |
| **Défaillance d'accès** | Rend le système indiscriminé : un client voit les données d'un autre |

---

## Les embeddings en clair

Un embedding = un texte converti en liste de nombres.

Exemple :
"Le patient a un antécédent de diabète type 2"
devient [0.82, 0.14, 0.91, ...]

**Vec2Text** reconstruit jusqu'à **92% des mots** de textes courts
à partir des vecteurs seuls, sans accès aux documents originaux.

Une fuite d'embeddings = une fuite de documents.
Les backups vectoriels ont le même niveau de sensibilité que les documents sources.

---

## Les 7 risques principaux

| Risque | Principe |
|---|---|
| **Cross-tenant leakage** | La recherche parcourt tout l'index avant le filtre par client |
| **Embedding inversion** | Les vecteurs volés sont reconvertis en texte original |
| **Retrieval-time poisoning** | Du contenu est conçu pour être récupéré à la place du bon |
| **Retrieval jamming** | Un document bloqueur empêche le système de répondre |
| **Membership inference** | L'attaquant déduit si un document existe sans voir son contenu |
| **Semantic cache poisoning** | Un cache sémantique sert du contenu attaquant à des requêtes équivalentes |
| **Multimodal poisoning** | Une image est conçue pour être récupérée en réponse à une requête texte |

---

## Le point clé sur le contrôle d'accès

Le filtrage par client doit se faire **à l'intérieur de la requête à l'index**,
pas après la recherche.

Si la recherche parcourt tout l'index puis filtre :
→ un client peut déduire l'existence et le sujet des données d'un autre
à partir des scores et des temps de réponse,
même sans jamais voir les documents.

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
| ⭐⭐⭐ | **Filtrer par client dans la requête à l'index**, pas après |
| ⭐⭐⭐ | **Traiter les backups vectoriels comme les documents sources** |
| ⭐⭐⭐ | **Ne jamais renvoyer les scores de similarité bruts** au client |
| ⭐⭐ | Séparer physiquement les index par niveau de confiance |
| ⭐⭐ | Nettoyer le contenu avant d'embedder (caractères invisibles, homoglyphes) |
| ⭐ | Logs immuables des activités de récupération |

---

## Liens avec les autres risques

- → **LLM02 Sensitive Info Disclosure** : inversion d'embeddings = fuite de données réglementaire
- → **LLM05 Data & Model Poisoning** : empoisonnement du modèle d'embedding lui-même
- → **LLM01 Prompt Injection** : l'injection indirecte via contenu récupéré est couverte par LLM01

---

## En une phrase
> Une base vectorielle mal configurée peut fuiter ses documents originaux
> sans que personne ne s'en aperçoive.
> Les embeddings ne sont pas des données anonymes.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM09*
