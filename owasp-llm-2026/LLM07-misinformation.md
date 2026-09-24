# LLM07:2026 — Misinformation

## C'est quoi ?
Le système produit une information fausse, incomplète ou trompeuse,
mais suffisamment crédible pour influencer une décision humaine,
un workflow automatisé ou une action d'agent.

Le vrai risque : ce n'est pas l'erreur en elle-même,
c'est qu'on la croit et qu'on agit dessus.

---

## Les causes

| Cause | Exemple |
|---|---|
| **Hallucination** | Le modèle invente un paquet, une loi, un cas juridique |
| **Contexte incomplet ou périmé** | Le modèle répond sur des données obsolètes |
| **Résumé trompeur** | Un point critique est omis dans le résumé |
| **Attaque délibérée** | L'attaquant injecte de fausses informations dans le contexte |
| **Propagation inter-agents** | Une erreur d'un agent est reprise comme vérité par un autre |

---

## Les formes spécifiques à un agent

**1. Déduction d'état incorrecte**
L'agent déduit qu'une condition est remplie alors qu'elle ne l'est pas
et déclenche une action.

**2. Propagation entre agents**
Un agent déclare un client comme vérifié alors qu'il ne l'est pas.
L'agent de paiement fait confiance et libère les fonds.

**3. Tâche faussement terminée**
L'agent affirme que la sauvegarde s'est bien passée.
Elle n'a jamais tourné. La restauration échoue.

**4. Omission critique**
Un résumé médical omet une contre-indication.
Le clinicien agit sur la base du résumé incomplet.

---

## Cas réels cités

- **Air Canada** : chatbot donnant de fausses informations sur les tarifs → poursuivi en justice
- **ChatGPT** : fabrication de cas juridiques inexistants utilisés devant un tribunal
- **Slopsquatting** : assistant code recommande un paquet inexistant → l'attaquant l'a enregistré avec du code malveillant

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Claim-check-act** : séparer la génération de l'exécution, vérifier avant d'agir |
| ⭐⭐⭐ | **Vérifier les préconditions et l'état réel** avant chaque appel d'outil |
| ⭐⭐⭐ | **Ne jamais faire confiance à l'affirmation d'un autre agent** sans vérification indépendante |
| ⭐⭐ | Sorties structurées avec champs obligatoires pour éviter les omissions |
| ⭐⭐ | RAG avec sources fiables et à jour pour ancrer les réponses |
| ⭐ | Distinguer les faits vérifiés des suppositions dans les sorties |

---

## Liens avec les autres risques

- → **LLM01 Prompt Injection** : une injection peut induire délibérément de la désinformation
- → **LLM05 Data & Model Poisoning** : des données corrompues produisent des outputs faux
- → **LLM10 Improper Output Handling** : différent — LLM07 = info fausse, LLM10 = sortie dangereuse pour le système

---

## En une phrase
> Le modèle peut produire des informations fausses avec une confiance totale.
> Ne laisse jamais un agent agir sur sa propre affirmation sans vérification indépendante.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM07*
