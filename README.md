# ARON et ARIA

Deux assistants IA pour les étudiants du BUT Réseaux et Télécoms de l'IUT RCC (Châlons-en-Champagne), hébergés sur une infrastructure locale.

| | **ARIA** | **ARON** |
|---|---|---|
| Nom | Assistante de Réflexion Intelligente et d'Aide | Assistant de Réalisation Optimisé Numérique |
| Rôle | Tuteur : aide l'étudiant à comprendre et à trouver par lui-même | Chef de projet et développeur sénior : réalise le travail dans les règles de l'art |
| Interface | Site web (OpenWebUI) | Extension Cline dans VS Code |
| Source des connaissances | Cours, TD et TP fournis par les professeurs (RAG) | Ses connaissances de développeur, guidées par un prompt de bonnes pratiques |
| Donne la réponse ? | Non : questions, indices, reformulations | Oui : il produit le code, la documentation et les tests |

## Sommaire

| Fichier | Contenu |
|---|---|
| [Contexte.md](Contexte.md) | D'où vient le projet et qui est concerné |
| [serveur_IA.md](serveur_IA.md) | Les GPU, le modèle, vLLM, LiteLLM, les prompts |
| [OpenWebUi.md](OpenWebUi.md) | L'interface web d'ARIA |
| [RAG.md](RAG.md) | La recherche dans les cours des professeurs |
| [site_acces_IA.md](site_acces_IA.md) | Le portail de gestion des accès (blocages, examens) |
| [plan_projet.md](plan_projet.md) | Étapes, budget, risques, RGPD |
| [prompts/ARIA.md](prompts/ARIA.md) | Prompt système d'ARIA et jeu de tests |
| [prompts/ARON.md](prompts/ARON.md) | Prompt système d'ARON |

## Les idées directrices

**Des IA fiables, pas un nouveau ChatGPT.** ARIA ne répond pas de mémoire : elle interroge la base de documents des professeurs et s'appuie sur elle. Les réponses viennent donc de sources approuvées, et elle indique d'où elles viennent.

**ARIA guide, ARON réalise.** ARIA pose des questions et donne des indices progressifs, sans livrer la solution d'un exercice. ARON, au contraire, produit un travail complet : documentation, gestion d'erreurs, typage, tests unitaires, code modulaire.

**Les professeurs gardent la main.** Chaque étudiant a ses propres accès. Les professeurs peuvent bloquer une personne, un groupe ou une salle, pour une durée donnée ou jusqu'à nouvel ordre, par exemple pendant un examen.

## Vue d'ensemble

```txt
+-------------------+  +-------------------+  +-------------------+
| Navigateur        |  | Navigateur        |  | VS Code + Cline   |
| (professeur)      |  | (étudiant / prof) |  | (étudiant)        |
+---------+---------+  +---------+---------+  +---------+---------+
          |                      |                      |
          v                      v                      |
+-------------------+  +-------------------+            |
|  PORTAIL D'ACCÈS  |->|     OpenWebUI     |            |
|  comptes, clés,   |  |  interface ARIA   |            |
|  blocages         |  |  + RAG des cours  |            |
+---------+---------+  +---------+---------+            |
          |                      |                      |
          v                      v                      v
+-----------------------------------------------------------------+
|                             LiteLLM                             |
|    clés API, ajout du prompt ARIA / ARON, répartition de charge |
+-----------------------------------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------+
|        vLLM + Qwen3.8 27B (4 bits), un conteneur par GPU        |
+-----------------------------------------------------------------+
```

Lecture du schéma :

1. L'étudiant parle à **ARIA** depuis le navigateur. OpenWebUI cherche dans les cours (RAG), puis envoie la question à LiteLLM avec la clé ARIA de l'étudiant.
2. L'étudiant travaille avec **ARON** depuis VS Code. Cline envoie ses requêtes directement à LiteLLM avec la clé ARON de l'étudiant.
3. **LiteLLM** vérifie la clé, ajoute le prompt système d'ARIA ou d'ARON, puis choisit un GPU disponible.
4. **vLLM** fait tourner le modèle et renvoie la réponse.
5. Le **portail d'accès** agit sur OpenWebUI (comptes) et sur LiteLLM (clés) pour bloquer ou débloquer des étudiants. Les professeurs ajoutent leurs cours dans OpenWebUI.

## Le modèle

Les deux IA utilisent le même modèle de base, **Qwen3.8 27B**, qui comprend le français et l'anglais, gère les outils et génère du code. Pour tenir sur une carte de 24 Go, il est utilisé quantifié en 4 bits (AWQ, environ 17 Go). La différence entre ARIA et ARON vient du **prompt système**, ajouté par LiteLLM, pas d'un entraînement différent. Détails dans [serveur_IA.md](serveur_IA.md).

## Le matériel

Cible : environ 70 étudiants (deux classes) qui peuvent utiliser les IA en même temps.

| Option | Description | Remarque |
|---|---|---|
| **A (architecture de référence)** | 8 à 10 cartes de 24 Go (type RTX 4090), un conteneur vLLM par carte | Environ 24 500 à 38 000 € avec les serveurs ; estimation de 2 à 4 sessions confortables par carte, soit 16 à 32 sessions en parallèle avec 8 cartes |
| **B** | 2 cartes de 48 Go (RTX 6000 Ada) ou H100 | Environ 19 000 à 29 000 € ; moins d'instances, mais chacune a plus de mémoire pour les longs contextes, et bien moins de consommation |

À ajouter dans les deux cas : une **petite carte dédiée de 8 à 12 Go** pour l'embedding et le reranker du RAG (voir [RAG.md](RAG.md)).

Ces capacités sont des **estimations**. 70 étudiants ne veulent pas dire 70 requêtes simultanées, mais ARON (Cline) envoie de gros contextes qui consomment beaucoup de mémoire. Un test de charge sur une seule carte doit confirmer les chiffres avant tout achat.

Le détail des prix, de la consommation et des risques est dans [plan_projet.md](plan_projet.md).

Les RTX 4090 sont des cartes grand public ; leur licence peut les interdire en datacenter. Cela se vérifie avant l'achat si le serveur est hébergé dans un établissement.

## Les examens

Chaque étudiant a deux clés API personnelles (une pour ARIA, une pour ARON). Le portail d'accès peut bloquer toutes les clés d'un groupe, d'une salle ou d'une personne pendant un examen. Les étudiants concernés n'ont alors plus accès ni à ARIA ni à ARON.

Cela réduit fortement les possibilités de tricher avec **nos** IA. Cela ne couvre pas les IA extérieures ni un téléphone personnel : c'est une mesure d'aide à la surveillance, pas une garantie.

## État du projet

Le dépôt ne contient pour l'instant que de la documentation. Rien n'est encore installé.

**À valider avant d'acheter le matériel** (détail dans [serveur_IA.md](serveur_IA.md)) :

1. Qwen3.8 27B en AWQ 4 bits démarre avec vLLM sur **une seule** carte de 24 Go.
2. Combien de requêtes en parallèle une carte supporte avant que la latence ne devienne gênante, avec le RAG actif.
3. L'injection des prompts par LiteLLM fonctionne avec Cline et OpenWebUI.
4. ARIA résiste aux tentatives de contournement (jeu de tests dans [prompts/ARIA.md](prompts/ARIA.md)).
5. Les API d'OpenWebUI et de LiteLLM permettent bien les blocages décrits dans [site_acces_IA.md](site_acces_IA.md).

**Étapes envisagées :**

1. Prototype sur une carte : vLLM, LiteLLM, OpenWebUI, ARIA avec RAG sur deux ou trois cours.
2. Test avec quelques étudiants et un professeur, mesure de la charge réelle.
3. Portail de gestion des accès.
4. ARON avec Cline.
5. Montée en charge (achat des cartes restantes).
