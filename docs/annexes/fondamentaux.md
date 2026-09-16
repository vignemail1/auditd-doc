# Fondamentaux de la sécurité Linux

## Trois mécanismes complémentaires

- **DAC** : permissions propriétaires, groupes, ACL et privilèges Unix ;
- **MAC** : politique imposée indépendamment du propriétaire, via AppArmor ou SELinux ;
- **audit** : enregistrement des événements pour investigation et conformité.

Un processus doit donc satisfaire les permissions Unix **et** la politique MAC. Auditd observe notamment les appels système et les changements de configuration, mais ne remplace pas une politique de contrôle d'accès.

## Prérequis communs

- accès root ou `sudo` ;
- sauvegarde et console hors bande ;
- synchronisation horaire ;
- stockage dimensionné pour les journaux ;
- environnement de test représentatif ;
- inventaire des services et flux réseau.

## Principes

1. Commencer par une politique minimale et explicite.
2. Modifier une règle à la fois.
3. Observer les refus et les faux positifs.
4. Versionner les changements.
5. Ne jamais résoudre durablement un incident par une désactivation globale.
