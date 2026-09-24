# LLM04:2026 — Supply Chain

## C'est quoi ?
Le danger ne vient pas toujours du modèle lui-même, mais de tout ce qui
a servi à le construire ou à le faire tourner : paquets, modèles
pré-entraînés, adaptateurs, datasets, services de conversion, pipelines CI/CD.

---

## Ce qui fait partie de la chaîne

| Composant | Exemple |
|---|---|
| **Paquets Python** | torch, transformers, langchain |
| **Modèles pré-entraînés** | Un modèle téléchargé sur Hugging Face |
| **Adaptateurs LoRA** | Un fine-tune rajouté sur un modèle de base |
| **Datasets** | Les données utilisées pour entraîner ou fine-tuner |
| **Services de conversion** | Un outil qui convertit un modèle d'un format à un autre |
| **Serveurs MCP** | Les outils que ton agent utilise |
| **Pipeline CI/CD** | Le système qui build et déploie ton modèle |

---

## Les attaques principales

**1. Slopsquatting**
L'assistant de code hallucine un nom de paquet inexistant.
L'attaquant l'a enregistré à l'avance avec du code malveillant.
Tu installes sans vérifier.

**2. Modèle piégé**
Un modèle publié sur Hugging Face passe les benchmarks mais contient
une porte dérobée. Elle s'active seulement sur un déclencheur précis,
invisible lors des tests normaux.

**3. Le format "sûr" qui ne l'est pas**
Pickle est dangereux (exécute du code au chargement), ONNX est réputé sûr.
Faux : une porte dérobée peut être cachée dans le graphe de calcul
d'un modèle ONNX sans aucun code exécutable à détecter.

**4. Quantification piégée**
Un modèle se comporte normalement en pleine précision.
Une fois quantifié, il exhibe un comportement malveillant.
Les tests faits sur la version complète ne détectent rien.

**5. Namespace reuse**
Un modèle est référencé par son nom Auteur/NomModele.
L'auteur supprime son compte. L'attaquant enregistre le même nom
et publie un modèle malveillant. Ton pipeline télécharge le mauvais automatiquement.

---

## Cas réels

- **PyTorch 2022** : paquet malveillant sur PyPI usurpant une dépendance PyTorch-nightly
- **PoisonGPT** : modèle avec paramètres modifiés diffusant de la désinformation tout en passant les tests
- **Ultralytics** : cache GitHub Actions empoisonné → releases PyPI piégées
- **Ollama CVE-2024-37032** : exécution de code à distance via un manifeste de modèle malveillant

---

## Mitigations clés

| Priorité | Mitigation |
|---|---|
| ⭐⭐⭐ | **Référencer les modèles par empreinte immuable**, jamais par un tag comme `latest` |
| ⭐⭐⭐ | **Vérifier que les paquets suggérés par une IA existent vraiment** avant de les installer |
| ⭐⭐⭐ | **SBOM étendu aux modèles et datasets** : inventaire signé de tous les composants |
| ⭐⭐ | **Signer les modèles** avec un outil comme Sigstore |
| ⭐⭐ | **Red teaming** sur les modèles tiers avant déploiement |
| ⭐ | Traiter les scanners comme une couche parmi d'autres, pas comme une garantie |

---

## Liens avec les autres risques

- → **LLM05 Data & Model Poisoning** : LLM04 = composant compromis dans la chaîne, LLM05 = données ou comportement corrompus
- → **LLM01 Prompt Injection** : les serveurs MCP font partie de la supply chain

---

## En une phrase
> Un composant compromis dans la chaîne compromet tout le système.
> Vérifie l'origine de tout ce que tu utilises.

---

*Source : OWASP Top 10 for LLM Applications 2026 — LLM04*
