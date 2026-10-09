# Le RAG d'ARIA

## Objectif

ARIA ne répond pas depuis sa seule mémoire : à chaque question elle interroge la base de documents fournie par les professeurs (cours, énoncés de TD/TP, évaluations d'entraînement) et construit son aide à partir de ces sources. Le RAG (Retrieval Augmented Generation) est ce mécanisme de recherche.

Il est intégré à OpenWebUI (fonction "Knowledge"), il n'y a donc pas de moteur de recherche à développer.

## Le principe

```txt
+---------------------------+
|     Question de l'élève   |
+-------------+-------------+
              |
              v
+---------------------------+
|        Embedding          |
|  question -> vecteur      |
+-------------+-------------+
              |
              v
+---------------------------+
|     Recherche hybride     |
|  vecteurs + mots-clés     |
|  (BM25) dans les cours    |
+-------------+-------------+
              |  20 à 50 candidats
              v
+---------------------------+
|        Reranker           |
|  garde les 3 à 5 meilleurs|
+-------------+-------------+
              |
              v
+---------------------------+
|  LiteLLM (modèle « aria »)|
|  + prompt d'ARIA          |
|  -> Qwen répond avec les  |
|     extraits et les       |
|     sources               |
+---------------------------+
```

L'indexation (découpage des documents en blocs puis calcul des vecteurs) se fait une seule fois, quand un professeur ajoute un document. Elle peut être lente sans gêner les élèves.

## Matériel

L'embedding et le reranker sont de petits modèles (environ 570 millions de paramètres chacun, contre 27 milliards pour Qwen). Ils ne doivent **pas** tourner sur les cartes du LLM, qui sont presque pleines (17 Go de poids sur 24 Go).

 - Une carte dédiée de 8 à 12 Go suffit pour les deux modèles (estimation à confirmer par un test).
 - Ils tournent dans leur propre conteneur, pour pouvoir être redémarrés sans toucher aux LLM.
 - Le gain de la carte graphique est surtout sensible pour **l'indexation** et pour le **reranker** (qui note 20 à 50 blocs par question). Pour l'embedding d'une seule question courte, le gain en temps est faible, même sur CPU.

## Réglages de départ

Ces valeurs viennent de guides tiers et de la documentation d'OpenWebUI. Aucune source ne donne de valeur propre au français, elles doivent être ajustées par des tests. Les noms exacts des réglages dépendent de la version d'OpenWebUI (Admin Panel > Settings > Documents).

| Réglage | Valeur de départ |
|---|---|
| Modèle d'embedding | `BAAI/bge-m3` (multilingue, supporte le français) |
| Reranker | `BAAI/bge-reranker-v2-m3` |
| Découpage | par tokens, environ 500 tokens par bloc |
| Chevauchement | 50 à 80 tokens |
| Découpage par titres Markdown | activé, avec fusion des petits fragments |
| Recherche hybride | activée, poids BM25 environ 0,5 |
| Candidats avant reranking | 20 à 50 |
| Blocs gardés après reranking (Top K) | 3 à 5 |

Changer de modèle d'embedding oblige à tout ré-indexer : il faut le choisir avant d'ajouter beaucoup de cours.

## Organisation de la base

 - Une **collection par cours** (ou par promotion), associée au modèle ARIA correspondant.
 - Les **corrigés** vont dans une collection séparée, réservée aux professeurs, qui n'est attachée à aucun modèle étudiant. Sinon ARIA peut retrouver la solution et la donner, ce qui casse son rôle de guide.
 - Les évaluations d'entraînement qui contiennent les réponses sont traitées comme des corrigés.
 - Si un professeur veut quand même qu'ARIA s'appuie sur un corrigé, il faut un prompt système strict ("ne révèle jamais la solution complète"). Un prompt n'est pas une garantie : à tester avec de vrais exemples.
 - Format des documents : les PDF issus de diaporamas ou de scans se découpent mal. Le Markdown ou le texte propre donne de meilleurs résultats. Format fourni par les professeurs : à définir.

## Citation des sources

ARIA doit indiquer le document et la section d'où vient chaque information, pour que les élèves et les professeurs puissent vérifier. À tester dans OpenWebUI avec la version utilisée.

## Protocole de test

1. Choisir 2 ou 3 vrais cours.
2. Préparer 15 à 20 questions dont la réponse est connue et se trouve dans les cours.
3. Changer **un seul** groupe de réglages à la fois (découpage, puis recherche hybride, puis reranker) et rejouer les questions.
4. Noter pour chaque question : le bon extrait est-il retrouvé ? la source est-elle citée ? ARIA donne-t-elle la solution complète au lieu de guider ?

## Points ouverts

 - Impact du RAG sur la capacité : 3 à 5 blocs de 500 tokens ajoutent environ 1 500 à 2 500 tokens par question dans le contexte. Avec 5 à 6 Go de cache KV par GPU, le nombre de sessions par carte (estimé à 2 à 4) est à revoir avec le RAG actif.
 - Droits d'accès : qui peut ajouter des documents à une collection, et comment un professeur gère ses cours.
 - Format et volume réels des documents des professeurs.
 - Vérification des performances de l'embedding et du reranker sur la carte dédiée.
