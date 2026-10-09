# OpenWebUI : l'interface d'ARIA

## Présentation

OpenWebUI est une interface web open source qui ressemble à ChatGPT : l'utilisateur se connecte avec son compte et discute avec une IA. Elle se branche sur n'importe quel serveur compatible avec l'API OpenAI, ce qui est le cas de LiteLLM.

Dans ce projet, OpenWebUI est l'interface d'**ARIA** uniquement. ARON, lui, s'utilise depuis VS Code avec Cline.

## Ce qu'elle apporte

**Pour l'administration :**

 - création, suppression et blocage de comptes ;
 - possibilité de renseigner une clé API pour chaque utilisateur ;
 - gestion des documents consultables par l'IA (« Knowledge », voir [RAG.md](RAG.md)).

**Pour l'étudiant :**

 - saisie vocale (transcription de la parole) ;
 - envoi de fichiers, y compris des archives zip, pour que l'IA les lise ;
 - réponses mises en forme, avec le code dans des blocs dédiés ;
 - génération de documents téléchargeables.

## Comment elle s'intègre

```txt
Étudiant ---> OpenWebUI ---> LiteLLM (modèle « aria ») ---> vLLM
                  |
                  +---> RAG : cours des professeurs
```

 - OpenWebUI envoie les questions à LiteLLM en demandant le modèle `aria`. LiteLLM ajoute le prompt système d'ARIA (voir [serveur_IA.md](serveur_IA.md)).
 - Avant d'envoyer la question, OpenWebUI retrouve les passages pertinents des cours (voir [RAG.md](RAG.md)) et les joint à la requête.
 - Chaque étudiant a sa propre clé ARIA dans LiteLLM, enregistrée dans son compte OpenWebUI.

## Contraintes à respecter

| Contrainte | Où elle est assurée |
|---|---|
| L'étudiant ne doit **jamais voir** sa clé API | Fonctionnement d'OpenWebUI (à vérifier dans la version utilisée) |
| Un compte bloqué ne doit **plus** pouvoir se connecter ni consulter ses anciennes conversations | Blocage du compte par le [portail d'accès](site_acces_IA.md) |
| L'étudiant ne doit pas pouvoir changer le prompt système d'ARIA | Le prompt est ajouté côté serveur par LiteLLM, pas dans OpenWebUI |

## À vérifier

 - La gestion d'une clé API par utilisateur dans la version d'OpenWebUI retenue.
 - Les API d'administration permettant au portail de bloquer et débloquer un compte (voir [site_acces_IA.md](site_acces_IA.md)).
 - L'affichage des sources citées par ARIA lorsqu'elle utilise le RAG.
