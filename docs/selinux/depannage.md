# SELinux — dépannage

## Refus AVC

```bash
sudo ausearch -m AVC -ts recent -i
sudo ausearch -m AVC -ts recent | audit2why
```

Vérifier d'abord les contextes :

```bash
ls -Zd /chemin
matchpathcon /chemin
sudo restorecon -v /chemin
```

## Service bloqué

Contrôler le port avec `semanage port -l`, les booléens pertinents et les permissions Unix. Tester l'action nominale après chaque correction.

## Éviter les mauvaises corrections

Ne pas désactiver SELinux ni appliquer un module généré sans revue. Un refus répétitif peut signaler une compromission, une erreur de configuration ou une politique incomplète : distinguer ces cas avant d'autoriser.