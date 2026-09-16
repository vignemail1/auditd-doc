# SELinux — vue d'ensemble

SELinux est un mécanisme MAC fondé sur des contextes et une politique de types. Les décisions dépendent notamment du domaine du processus, du type de la ressource et de la classe/permission demandée.

## Fonctionnalités

- séparation des domaines de processus ;
- étiquetage des fichiers, ports et objets ;
- booléens pour options contrôlées ;
- politiques modulaires ;
- journalisation des refus AVC.

## Contraintes

SELinux demande une compréhension des contextes et des transitions. Modifier directement les contextes ou générer des règles sans analyse peut masquer un défaut de conception. Le mode permissif est destiné au diagnostic, pas à l'exploitation durable.