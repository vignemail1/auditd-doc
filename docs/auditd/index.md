# auditd — vue d'ensemble

`auditd` collecte les événements du sous-système d'audit du noyau : appels système, accès à des fichiers, changements d'identité, exécution de programmes et événements de sécurité.

## Fonctionnalités

- traçabilité avec identités réelle et effective ;
- filtrage par architecture, syscall, utilisateur, fichier et clé ;
- rapports et recherches historiques ;
- règles persistantes ;
- relais vers un collecteur distant.

## Contraintes

Auditd ne bloque pas un accès. Des règles trop nombreuses augmentent le volume et le coût CPU. Les journaux peuvent contenir des données sensibles : protégez-les et dimensionnez leur stockage.

## Commandes principales

`auditctl`, `augenrules`, `ausearch`, `aureport`, `auditd`, `audisp-remote`.