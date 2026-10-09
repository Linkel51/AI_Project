# ARON ET ARIA

- ARON : Assistant de Réalisation Optimisé Numérique
- ARIA : Assistante de Réflexion Intelligente et d'Aide

Deux IA, deux philosophies, la ou ARON est une ia faite pour être une tête pensante de projet, une IA qui pense a tout, qui fait tout correctement, ARIA est une IA faite pour aider les étudiants à réfléchir, à répondre au questions qui leurs sont posées sans pour autant leur donner la réponse imédiatement, mais plutôt les guider vers la solution par le biais de questions et d'indices.

## Des IA connectées

Ces deux IA sont conçus pour ne pas être de nouveaux chatgpt, elles sont reliées directement à un moteur de recherche puissant, leurs permettant d'intéroger une base de données fournis par les professeurs de la formation, l'idée c'est que les réponses qu'elles apportent sont fiables et approuvées par les professeurs, elles sont le vecteur de leur savoirs, un élève pose une question à ARIA, ARIA intéroge la base de données du cours en liens et répond à l'élève de cette manière on s'assure de manière maximale de la provenance des informations.

## ARIA : une professeure

ARIA n'est pas une IA classique dans l'apprentissage, il s'agit avant tout d'une IA qui si on lui pose une question, va aider vraiment l'élève a construire un raisonnement, à réfléchir par lui même, dans cette optique, ARIA ne va pas donner la réponse directement, mais plutôt poser des questions à l'élève pour l'aider à trouver la solution par lui même, elle va aussi lui donner des indices pour l'aider à avancer dans sa réflexion.

## ARON : un chef de projet et un ouvrier sénior

ARON est une IA faite pour être directement intégré a des outils a l'aide de programme comme CLINE, il est fait pour être directement intégrer a des outils pour travailler sur des projets. Il ne se contente pas de faire le travail de manière bête, mais plutot de le faire en respectant un maximum toute les bonnes pratiques, via une vrai documentations, des gestion d'erreurs, une définitions des types, des test unitaires, une modularité dans la programmations, et autres.

## Sous le capot

Ces deux IA fonctionne avec le modèle Qwen 3.8 a 27 milliards de paramètres, c'est un modèle de langage qui comprend le français et l'anglais, capables de comprendre du texte de générer des réponses structurées pour fonctionner avec des outils, et de générer du code complexe avec de gros contextes.

Pour tenir sur des cartes de 24 Go, le modèle est utilisé quantifié en 4 bits (AWQ, environ 17 Go). Voir [Le serveur IA](<Le serveur IA>) pour le détail et les points restant à valider.

## Les examins 

Lors des examens, comme chaque étudiant aura une api qui lui sera propre, les API des étudiants appartenant à un groupe donné pourront être bloquées, de cette manière les étudiants ne pourront pas s'aider d'ARON et n'auront pas accès à ARIA, les api s'accompagneront d'une mot de passe obligatoire, de cette manière il sera très difficile pour un étudiant d'utiliser ces deux IA pour tricher pendant un examen (le blocage ne couvre pas les IA externes ou un téléphone personnel).

## Le matériel

Pour 70 étudiants (ce qui correspond à deux classes à peu de choses près) qui utilise en simultané les deux IA, on peut estimer qu'il peut être nécessaire d'avoir pluesieurs instances de Qwen 3.8 a 27 milliards de paramètres, pour que les deux IA puissent fonctionner correctement et répondre aux demandes des étudiants sans ralentissement. Il y aurait donc deux possibilités, soit il faudrait deux cartes graphiques Nvidia optimisées pour l'IA, avec 48Go de VRAM, les Nvidia RTX 6000 Ada ou les Nvidia H100 sont des cartes graphiques qui peuvent convenir pour ce genre de projet, soit il faudrait entre 8 et 10 carte graphique plus petite comme des 4090 de 24Go chaucune pour que chacune puisse posséder une IA, ce qui ferait un budget d'environ 20 000€ pour le matériel. Avec le modèle quantifié en 4 bits, on estime 2 à 4 sessions confortables par carte, soit environ 16 à 32 sessions en parallèle avec 8 cartes, ce qui est suffisant pour 70 étudiants car ils n'envoient pas tous une requête en même temps. Ce chiffre doit être confirmé par un test de charge sur une carte avant l'achat.

## L'infrastructure

```txt
+--------------------+
|     vLLM avec      |
|   2*48Go de VRAM   |
+---------+----------+
          ⬇
+---------+----------+
|        CUDA        |
+---------+----------+
          ⬇
+---------+----------+                +--------------------+
|    qwen 3.8 27b    |                |   BASE DE DONNEE   |
+----+-----------+---+                |     DES PROFS      |
     |           |                    +---------+----------+
     |           |                              ⬇
     |           |                    +---------+----------+
     |           |                    |     NAVIGATEUR     |
     |           |                    +---------+----------+
     |           |                              ⬆
     |           |                    +---------+----------+
     |           |                    |   AUTENTIFICATION  |
     |           |                    +---------+----------+
     |           ⬇                   ↗          ⬆
     |         +-+------------------+           |
     |         |   INTERFACE WEB    |           |
     |         |        ARIA        |           |
     |         +--------------------+           |
     |                                          |
     |                                +---------+----------+
     |                                | SITE DE GESTION    |
     |                                | DES ACCES          |
     |                                | UTILISATEUR (PROF) |
     |                                +---------+----------+
     |                                          |
     |         +--------------------+           |
     +-------➡|    GESTION DES     +           |
               |        API         |           |
               +---------+----------+           |
                         |                      ⬇
                         |            +---------+----------+
                         +----------➡|        API         |
                                      |   AUTENTIFICATION  |
                                      +---------+----------+
                                                ⬇
                                      +---------+----------+
                                      |        CLINE       |
                                      +--------------------+
```
