# Embedding Inversion Lab 🔐

Démonstration pratique d'une **embedding inversion attack** 
sur une base vectorielle RAG contenant des dossiers médicaux fictifs.

Basé sur **LLM09 — Vector and Embedding Weaknesses** 
de l'OWASP Top 10 for LLM Applications 2026.

---

## C'est quoi une embedding inversion attack ?

Un embedding c'est un texte converti en liste de nombres :

"Le patient a un antécédent de diabète type 2"
→ [-0.00192893, 0.07245513, -0.01145829, ...]


On pense que ces chiffres sont abstraits et inoffensifs.

**C'est faux.**

Une attaque d'inversion reconstruit le texte original
depuis ces chiffres, sans jamais avoir accès aux documents.

Vec2Text reconstruit jusqu'à **92% des mots exacts**.

➡ Une fuite d'embeddings = une fuite de documents.

---

## Ce que ce lab démontre

1. Créer une base vectorielle RAG avec des dossiers médicaux fictifs
2. Simuler une fuite des embeddings
3. Lancer une attaque d'inversion par similarité cosinus
4. Tester les mitigations et mesurer leur efficacité

---

## Résultats

| Scénario | Documents reconstruits |
|---|---|
| Sans protection | 4/5 (80%) |
| Avec bruit sur les embeddings | 4/5 (80%) — insuffisant seul |
| Avec contrôle d'accès | 2/5 accessibles — 3 bloqués |

**Conclusion : le contrôle d'accès est la mitigation la plus efficace.**

---

## Stack technique

- Python 3
- ChromaDB — base vectorielle
- sentence-transformers — modèle all-MiniLM-L6-v2
- NumPy — calcul de similarité cosinus
- Jupyter Notebook

---

## Installation

```bash
pip install chromadb sentence-transformers jupyter numpy
jupyter notebook
```

Ouvre `embedding-inversion-lab.ipynb` et exécute les cellules dans l'ordre.

---

## Mitigations OWASP

| Mitigation | Efficacité | Description |
|---|---|---|
| Ajout de bruit | ⭐⭐ | Brouille les embeddings mais insuffisant seul |
| Contrôle d'accès | ⭐⭐⭐ | Chaque utilisateur accède seulement à ses données |
| Chiffrement de la base | ⭐⭐⭐ | La base vectorielle n'est jamais exposée directement |
| Pas de scores bruts en API | ⭐⭐ | L'attaquant ne peut pas affiner ses suppositions |

---

## Références

- [OWASP Top 10 for LLM Applications 2026](https://genai.owasp.org)
- LLM09 — Vector and Embedding Weaknesses
- LLM02 — Sensitive Information Disclosure
- Vec2Text — Morris et al. 2023

---

## Auteure

**Becky Joyce ELIBE**
Étudiante ingénieure — Réseaux & Cybersécurité — EFREI Paris
[LinkedIn](https://linkedin.com/in/becky-joyce-elibe10) | 
[GitHub](https://github.com/Beckyyjw)

---

*Tous les documents utilisés dans ce lab sont fictifs.*
*Aucune donnée réelle n'a été utilisée.*
