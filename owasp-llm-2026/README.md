# OWASP Top 10 for LLM Applications 2026

Notes d'étude personnelles basées sur le document officiel
**OWASP Top 10 for LLM Applications 2026 (v1.0 — août 2026)**.

Source : [genai.owasp.org](https://genai.owasp.org)

---

## Principe fondamental

> "Ne cherche pas à construire un modèle impossible à tromper.
> Construis le système autour du modèle pour que,
> le jour où il sera trompé, rien de grave ne casse."
>
> — OWASP GenAI Project Leads, 2026

---

## Les 10 risques

| # | Risque | Résumé |
|---|---|---|
| [LLM01](./LLM01-prompt-injection.md) | Prompt Injection | Une entrée malveillante modifie le comportement du modèle |
| [LLM02](./LLM02-sensitive-info-disclosure.md) | Sensitive Information Disclosure | Le système expose des données confidentielles |
| [LLM03](./LLM03-excessive-agency.md) | Excessive Agency | L'agent a trop de capacités, permissions ou autonomie |
| [LLM04](./LLM04-supply-chain.md) | Supply Chain | Des composants tiers compromis dans la chaîne |
| [LLM05](./LLM05-data-model-poisoning.md) | Data and Model Poisoning | Corruption durable des données ou du modèle |
| [LLM06](./LLM06-unbounded-consumption.md) | Unbounded Consumption | Consommation de ressources sans contrôle |
| [LLM07](./LLM07-misinformation.md) | Misinformation | Information fausse suffisamment crédible pour influencer une action |
| [LLM08](./LLM08-hidden-context-exposure.md) | Hidden Context Exposure | Extraction du contexte caché (system prompt, outils, règles) |
| [LLM09](./LLM09-vector-embedding-weaknesses.md) | Vector and Embedding Weaknesses | Attaques sur la géométrie des embeddings et le RAG |
| [LLM10](./LLM10-improper-output-handling.md) | Improper Output Handling | Sortie du modèle transmise sans validation ni nettoyage |

---

## Les distinctions clés à ne pas confondre

| Paire | Différence |
|---|---|
| LLM01 / LLM05 | Instruction reçue à l'inférence / corruption durable |
| LLM01 / LLM09 | Exploite le suivi d'instructions / exploite la géométrie des embeddings |
| LLM02 / LLM08 | Fuite de données (PII, métier) / fuite du contexte de contrôle |
| LLM03 / LLM10 | Droits et autonomie trop larges / sortie non validée |
| LLM04 / LLM05 | Composant tiers compromis / données ou comportement corrompus |
| LLM07 / LLM10 | Information fausse / sortie dangereuse pour le système |

---

## Les 3 règles qui reviennent partout

**1. Moindre privilège**
L'agent n'a que les droits strictement nécessaires.
Lecture seule si lecture suffit. Jamais de compte admin par défaut.

**2. La Rule of Two (Meta, 2025)**
Un agent ne doit pas avoir plus de 2 des 3 capacités suivantes :
- A — lire du contenu non fiable
- B — accéder à des données sensibles
- C — agir vers l'extérieur

Si les 3 sont nécessaires → validation humaine obligatoire pour chaque action.

**3. Zero Trust sur les sorties du modèle**
Traiter chaque sortie du modèle comme une entrée utilisateur non fiable.
Valider, encoder, paramétrer avant d'agir.

---

## Frontière avec le Top 10 Agentic

Ce document couvre le modèle comme **composant** d'une application.
Dès que le modèle devient un **acteur autonome** (outils, mémoire persistante,
actions en cascade), le risque relève du
[OWASP Top 10 for Agentic Applications](https://genai.owasp.org).

Les deux listes sont complémentaires.

---

## Contexte

Ces fiches ont été rédigées dans le cadre d'un projet de
**sécurité d'agent IA**, en utilisant l'OWASP Top 10 LLM 2026
comme ressource principale.

---

*Dernière mise à jour : septembre 2026*
