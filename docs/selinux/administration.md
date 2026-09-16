# SELinux — administration et analyse

```bash
sudo sestatus
sudo semodule -l
sudo semodule -B
sudo restorecon -Rv /etc/myapp
sudo ausearch -m AVC -ts recent -i
sudo journalctl -t setroubleshoot
```

## Analyse d'un AVC

Examiner le domaine source, le type cible, la classe, la permission, le chemin, le contexte et le processus. Vérifier d'abord : mauvais contexte, port non déclaré, booléen pertinent, permission Unix ou application mal configurée.

## Modes de déploiement

Utiliser permissif temporairement pour recueillir des preuves, puis revenir à enforcing. Un redémarrage ou un réétiquetage complet doit être planifié, surtout après restauration d'une sauvegarde ou déplacement de volumes.