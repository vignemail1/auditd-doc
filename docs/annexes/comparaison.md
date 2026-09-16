# Comparaison et complémentarité

| Besoin | auditd | AppArmor | SELinux |
|---|---:|---:|---:|
| Tracer une action | Excellent | Indirect | Indirect |
| Restreindre un programme | Non | Oui, par chemin | Oui, par contexte |
| Contrôle du domaine d'accès | Non | Limité | Très riche |
| Prise en main rapide | Élevée | Élevée | Plus exigeante |
| Dépendance au chemin du binaire | Non | Oui | Non principalement |

## Choix pratique

- utiliser **auditd** pour la traçabilité et l'investigation ;
- préférer **AppArmor** si la distribution fournit des profils simples à maintenir et que l'approche par chemin convient ;
- préférer **SELinux** pour des politiques de domaines, des environnements multi-services ou des exigences fortes de séparation.

Ils peuvent fonctionner simultanément. Évitez toutefois de multiplier les règles redondantes sans objectif mesurable.