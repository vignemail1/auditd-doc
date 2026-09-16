# Sécurité Linux : auditd, AppArmor et SELinux

Ce guide s'adresse aux administrateurs Linux qui doivent **observer**, **comprendre** et **réduire** les risques sur des serveurs et postes de travail.

## Objectifs

- tracer les actions importantes avec **auditd** ;
- limiter les capacités des programmes avec **AppArmor** ou **SELinux** ;
- diagnostiquer les refus sans désactiver les protections ;
- déployer des règles maintenables et vérifiables.

> Les exemples sont à adapter à la distribution et à la version installées. Testez toujours une politique en préproduction.

## Parcours recommandé

1. Lire les [fondamentaux](fondamentaux.md).
2. Déployer l'audit avant de restreindre un service.
3. Étudier AppArmor ou SELinux selon la distribution.
4. Appliquer la [méthodologie](methodologie.md), puis automatiser.

## Avertissement

Une règle incorrecte peut bloquer un service ou produire des journaux volumineux. Conservez un accès console ou une procédure de retour arrière.