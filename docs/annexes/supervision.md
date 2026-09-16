# Supervision, conformité et réponse aux incidents

## Indicateurs

- service actif et règles chargées ;
- taux de refus AppArmor/SELinux ;
- volume et âge des journaux audit ;
- espace disque restant ;
- échecs de transmission ;
- modifications des politiques.

## Centralisation

Transmettre les événements vers une collecte protégée, synchroniser l'heure, limiter les accès et vérifier l'intégrité. Une copie locale ne doit pas être la seule preuve lors d'une compromission.

## Incident

1. préserver les journaux ;
2. noter heure, hôte et périmètre ;
3. rechercher les changements de privilèges et de politiques ;
4. contenir sans détruire les preuves ;
5. corriger puis vérifier ;
6. documenter les leçons apprises.

Les exigences de conservation doivent être définies avec les équipes conformité et juridiques.