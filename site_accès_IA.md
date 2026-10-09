# Le portail de gestion des accès ia

## L'infra

```txt
+-------------------------+
|    L'application web    |
+-------------------------+
      |             |   
+-----------+ +-----------+
|  liteLLM  | | OpenWebUI |
+-----------+ +-----------+
```

L'application WEB est directement connecter a liteLLM et OpenWebUI grace a leur API pour leur gestion.

## Les fonctionnalitées

Ce site devra :
 - offrir une gestion complète des comptes utilisateur
   - chaque utilisateur aura ses accès au service web ou a l'api agentique
   - il permet une gestion administrative glisser des comptes des deux autre système (réinitialiser le mot de passe etc...)
 - il devra aussi permettre de gérer un dashboard connecter a l'emploie du temps permettant :
   - un blocage temporaire de tout les comptes dont les personnes sont dans tel salle ou tel cours dans l'emploie du temps pendant x temps ou jusqu'au changement manuel sur une ou les deux IA (ARIA - interface web et ARON connecteur ia)
   - un blockage temporaire de tout les comptes appartenant a un groupe ici aussi pendant x temps ou jusqu'a indication contraire et de la même manière sur l'une ou l'autre des deux ia
   - un blocage ciblé sur une personne de la même manière pendant x temps ou jusqu'à instructions contraire et sur l'une ou les deux IA
