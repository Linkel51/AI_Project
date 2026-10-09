# Le portail de gestion des accès

## À quoi il sert

Le portail est l'outil des professeurs et des administrateurs pour gérer qui peut utiliser ARIA et ARON, et à quel moment. Son cas d'usage principal : **bloquer les IA pendant un examen**, sans avoir à intervenir à la main sur chaque compte.

## Où il se place

```txt
+---------------------------+
|     Portail d'accès       |
+------+-------------+------+
       |             |
       v             v
+-------------+  +-------------+
|   LiteLLM   |  |  OpenWebUI  |
| (clés API)  |  |  (comptes)  |
+-------------+  +-------------+
```

Le portail ne fait pas tourner d'IA. Il pilote les deux systèmes existants par leurs API d'administration :

 - **LiteLLM** : les clés API (une pour ARON, une pour ARIA, par étudiant) ;
 - **OpenWebUI** : les comptes de l'interface web d'ARIA.

## Fonctionnalités

### 1. Gestion des comptes

 - un compte unique par étudiant côté portail, avec ses accès à ARIA et/ou à ARON ;
 - création, suppression et réinitialisation de mot de passe, répercutées sur LiteLLM et OpenWebUI ;
 - import par groupe (promotion, TD, TP).

### 2. Blocages

Trois niveaux de blocage, chacun applicable à ARIA, à ARON ou aux deux :

| Cible | Exemple |
|---|---|
| **Une personne** | Un étudiant dont on suspend l'accès |
| **Un groupe** | Toute la promotion pendant un examen |
| **Une salle ou un cours** | Tous les étudiants inscrits dans la salle B12 de 14h à 16h, d'après l'emploi du temps |

Chaque blocage a une durée : **pendant X temps** ou **jusqu'à levée manuelle**.

### 3. Tableau de bord lié à l'emploi du temps

Le portail lit l'emploi du temps pour proposer les blocages au bon moment : un professeur choisit une épreuve dans la liste, et les groupes concernés sont bloqués pour la durée prévue.

## Comment les blocages sont appliqués

| Où | Moyen | Remarque |
|---|---|---|
| Clé API LiteLLM | Appel à `/key/block` pour bloquer, `/key/unblock` pour débloquer | D'après des guides tiers ; à vérifier dans la documentation officielle et le Swagger (`/docs`) de la version installée |
| Compte OpenWebUI | API d'administration d'OpenWebUI | Non vérifié : à confirmer pour la version retenue |

LiteLLM ne semble pas proposer de blocage « jusqu'à telle heure » : le portail doit donc gérer lui-même la durée. Il bloque au début de la période, puis un planificateur (tâche programmée) débloque à l'heure prévue.

Ce qui doit se passer pour un compte bloqué :

 - impossible de discuter avec ARIA ou ARON ;
 - impossible de consulter ses anciennes conversations sur OpenWebUI ;
 - les clés API ne fonctionnent plus.

## Points à clarifier

 - **Source de l'emploi du temps** : export ICS, ENT, ou autre. À demander à l'équipe pédagogique ; sans elle, les blocages « par salle » ne sont pas réalisables.
 - **Qui est autorisé à bloquer** : tous les professeurs, ou seulement certains ? Faut-il un journal des actions ?
 - **Fiabilité** : que se passe-t-il si le portail tombe en plein examen ? Prévoir une procédure de secours (blocage manuel dans LiteLLM).
 - **Données personnelles** : le portail manipule des identités d'étudiants et des emplois du temps (RGPD).
