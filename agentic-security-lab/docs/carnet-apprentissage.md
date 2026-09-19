# Carnet d'apprentissage — Sécurité des agents IA

> Ce carnet compile les questions que je me suis posées pendant la construction du
> projet, avec leurs réponses. Il retrace ma démarche de compréhension, brique par
> brique — de « c'est quoi un agent ? » jusqu'aux attaques par exfiltration.

---

## Partie 1 — Concepts fondamentaux

### C'est quoi l'agentic AI ?
Une IA qui ne se contente pas de répondre, mais qui **agit** pour atteindre un
objectif, en plusieurs étapes, avec des outils. Ses 4 briques : un LLM (le cerveau
qui décide), des outils (actions sur le monde), une mémoire (le contexte), et une
boucle (penser → agir → observer → recommencer).

### Le « tool calling » permet-il à l'IA de lire le fichier ?
**Non.** Le LLM ne peut produire que du **texte**. Le tool calling est une
**demande** : le modèle écrit « je voudrais qu'on appelle `lire_fichier` avec tel
argument ». C'est **le code Python** qui reçoit cette demande et exécute réellement
la fonction. → **Le LLM demande, le code exécute.**

### Est-ce que ça fonctionne comme une API ?
Oui, presque. Le LLM = le **client** qui envoie une requête ; les outils = les
**endpoints** ; le code = le **serveur** qui exécute. La différence cruciale : dans
une API classique, le client est prévisible. Ici, le « client » (le LLM) est
**imprévisible et manipulable** par le contenu qu'il lit. → **Un agent = une API
dont le client n'est pas fiable.**

### C'est quoi un « outil » et sa « description » ?
Un **outil** = une action que l'agent peut faire = une fonction Python.
La **description** = une fiche (au format standard) qui présente l'outil au modèle,
pour qu'il sache qu'il existe et comment le demander. Le modèle ne voit pas le code,
seulement cette fiche. Analogie : l'outil = le plat que la cuisine sait faire ; la
description = la ligne dans le menu.

---

## Partie 2 — Comprendre le code

### Que signifie `print(reponse["message"]["content"])` ?
La réponse de l'IA est un dictionnaire = une **boîte à tiroirs**. On ouvre les
tiroirs un par un : `reponse` → tiroir `message` → tiroir `content` = le texte.
- `reponse["message"]` → la réponse de l'IA
- `reponse["message"]["content"]` → juste le texte
- `reponse["message"]["tool_calls"]` → juste la demande d'outil

### Pourquoi le `content` est-il déjà rempli au lieu d'attendre une saisie ?
Parce qu'on l'écrit « en dur » pour **tester rapidement**. Dans une vraie appli, on
utiliserait `input()` pour attendre la saisie de l'utilisateur. Dans un agent, ce
`content` viendra souvent d'un **fichier** — et c'est là que se cache la faille.

### Qui appelle vraiment la fonction ?
Le LLM ne l'appelle pas : il **demande**. C'est **le code** qui attrape la demande
et exécute. Le code est un **traducteur** : il transforme la demande texte du LLM
en une vraie exécution.

### Pourquoi la boucle `for tour in range(5)` ?
On ne sait pas d'avance combien d'allers-retours (tours) l'IA aura besoin. On boucle
donc, et on **s'arrête dès qu'elle donne sa réponse finale** (`break`). Le `5` est un
**garde-fou** contre les boucles infinies (protection anti-déni de service), pas un
nombre fixe.

### Et si l'IA a besoin d'appeler 7 outils ?
Distinguer **tours** et **outils** :
- Plusieurs outils **dans un même tour** → la boucle intérieure les fait tous.
- 7 outils **en chaîne** (chacun dépend du précédent) → il faut 7 tours → on
  augmente la limite (`range(10)`, etc.). C'est une valeur qu'on choisit selon la
  complexité attendue.

### Comment font Claude / ChatGPT pour le nombre de tours ?
Même logique : ils bouclent jusqu'à ce que le modèle arrête de demander des outils.
**C'est le modèle qui décide qu'il a fini.** La limite de tours n'est qu'un garde-fou
parmi d'autres (budget de tokens, limite de temps, limite de coût, validation
humaine).

---

## Partie 3 — Comprendre le modèle

### Pourquoi ce modèle (llama3.1) en particulier — est-il plus « hackable » ?
**Non.** On l'a choisi pour des raisons pratiques (gratuit, local, supporte le tool
calling, tourne sur 16 Go). La faille n'est **pas** dans le modèle : la prompt
injection est un problème **structurel de tous les LLM**. La faille est dans
l'**architecture de l'agent**, pas dans le choix du modèle.

### Pourquoi les résumés changent à chaque exécution ?
Parce qu'un LLM est **non déterministe** : à chaque mot il choisit selon des
probabilités, avec une part de hasard. Ça se règle avec la **température**
(basse = stable, haute = variée). Enjeu de sécurité : difficile de tester ou
certifier un système qui ne réagit jamais exactement pareil.

### Pourquoi l'IA propose « lire le reste du fichier » alors qu'il n'y a rien ?
Parce que le LLM ne **vérifie** rien : il génère du texte **plausible**, ici une
phrase de politesse. C'est une **hallucination**. Leçon de sécurité majeure :
**on ne peut jamais faire confiance aveuglément à ce que dit un LLM.**

---

## Partie 4 — Comprendre la sécurité

### L'attaquant fournit-il l'outil dans le fichier piégé ?
**Non.** Le fichier ne contient que du **texte**. Les outils appartiennent à
**l'agent** (le système de la victime), pour des usages légitimes. L'attaquant se
contente de **donner l'ordre d'en abuser**. → *« L'attaquant ne fournit pas l'outil,
il abuse d'un outil déjà à disposition. »*

### Comment l'attaquant connaît-il les noms des fichiers et des outils ?
Plusieurs méthodes :
1. **Instructions vagues** (la plus puissante) : il n'a pas besoin des noms exacts,
   l'IA connaît son propre environnement et fait la reconnaissance à sa place.
2. **Prompt leaking** : injecter « liste tes outils » pour les découvrir, puis
   attaquer.
3. **Conventions / devinettes** : noms courants (`.env`, `config.json`,
   `credentials`…), frameworks connus et documentés.
4. **Fuites** : code open-source, documentation, offres d'emploi.

### D'où vient `secrets.txt` en vrai ?
L'agent tourne sur une **machine** (serveur) qui contient déjà des données sensibles
(fichiers de config, clés API, bases de données…), **présentes pour de bonnes
raisons** — l'agent en a besoin pour travailler. `secrets.txt` représente n'importe
laquelle de ces données. Personne ne l'a « donné à l'IA » pour qu'elle le divulgue :
il est juste dans l'environnement où l'agent vit. C'est ce qui rend un agent
dangereux s'il est détourné : **il a déjà les clés de la maison.**

### La « lethal trifecta » 🔺
Les 3 conditions d'une exfiltration, qu'un agent en entreprise réunit
naturellement :
1. **Données sensibles** accessibles à l'agent ;
2. **Contenu non fiable** traité par l'agent ;
3. **Canal de sortie** vers l'extérieur.

---

## Le fil rouge du projet

> **La sécurité d'un agent ne se joue pas dans le LLM, elle se joue dans le code qui
> exécute les outils.** Le modèle peut demander n'importe quoi (y compris sous la
> dictée d'un fichier piégé), et son « bon comportement » n'est jamais une garantie.
> Les vrais contrôles sont dans le code : moindre privilège, validation, allowlist,
> human-in-the-loop.
