# auditd — dépannage

## Aucun événement

Vérifier `auditctl -s`, l'état du service, la règle chargée, l'architecture, la période recherchée et les permissions du journal.

## Trop d'événements

Réduire les règles larges, ajouter des filtres, préférer une clé ciblée et mesurer les appels fréquents avant de surveiller globalement un répertoire.

## Disque plein

Contrôler `auditd.conf`, les actions d'espace, la rotation et la collecte distante. Ne supprimez pas les preuves avant validation de la procédure d'incident.

## Règle refusée

Chercher une erreur de syntaxe ou une option incompatible avec la version. Comparer les règles persistantes et celles actives avec `auditctl -l`.