# Automatisation et industrialisation

## Versionnement

Versionner les règles auditd, profils AppArmor et modules SELinux avec leur justification, leur périmètre et leur procédure de retour arrière.

## Contrôles CI

- validation syntaxique ;
- recherche de doublons et règles trop larges ;
- test dans une machine éphémère ;
- vérification du démarrage et des journaux ;
- contrôle qu'aucun secret n'est présent.

## Ansible

Utiliser des rôles séparés pour l'installation, la configuration et les politiques. Déclarer les fichiers avec permissions explicites, recharger uniquement après changement et prévoir `notify` pour les services.

## Déploiement progressif

Commencer par un groupe pilote, comparer les refus et les métriques, puis élargir. Ne pas mélanger une mise à niveau applicative et une restriction de politique sans fenêtre de diagnostic.