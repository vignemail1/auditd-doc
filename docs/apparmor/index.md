# AppArmor — vue d'ensemble

AppArmor applique une politique associée au chemin d'un programme. Un profil décrit les fichiers, capacités, signaux, réseaux et abstractions autorisés.

## Forces

- approche lisible et orientée application ;
- profils activables indépendamment ;
- modes complain et enforce ;
- intégration forte sur Ubuntu et Debian.

## Contraintes

Le chemin du programme est important ; les changements de chemin ou certains scénarios complexes demandent une conception attentive. Un profil incomplet peut interrompre un service ; un profil trop large apporte peu de réduction de risque.

Les refus sont généralement visibles dans le journal noyau et peuvent aussi être corrélés par auditd.