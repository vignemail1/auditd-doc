# Configuration d’auditd

Cette page décrit la configuration du démon `auditd` via `/etc/audit/auditd.conf`. Les règles d’audit (`auditctl`, `audit.rules`) sont documentées dans [Règles Auditd](regles.md).

## Rôle et structure du fichier

`auditd.conf` est un fichier texte composé d’une directive par ligne, sous la forme :

```text
clef = valeur
```

Les espaces autour de `=` sont facultatifs. Les lignes vides et les lignes commençant par `#` sont ignorées. Les noms de clés sont généralement insensibles à la casse selon la version ; utiliser les noms documentés en minuscules reste recommandé. Une clé ne doit apparaître qu’une seule fois : en cas de doublon, le comportement peut dépendre de l’implémentation.

Le fichier configure le démon : emplacement des journaux, rotation, comportement en cas de saturation ou d’erreur, et intégration avec `syslog` ou `systemd-journald`. Il ne définit pas les événements à surveiller.

Fichier habituel :

```text
/etc/audit/auditd.conf
```

Afficher la configuration effectivement lisible :

```bash
sudo auditd -s
sudo auditd -f
man auditd.conf
```

> Selon la distribution, `auditd -s` et les outils de paquet peuvent appliquer des contrôles supplémentaires. Vérifiez toujours la documentation de votre version.

## Paramètres principaux

### Journaux et format

| Clé | Valeurs attendues | Usage |
|---|---|---|
| `log_file` | chemin absolu | Fichier principal des journaux, par exemple `/var/log/audit/audit.log`. Le répertoire doit exister et être accessible par `auditd`. |
| `log_format` | `RAW` ou `ENRICHED` | `RAW` conserve les valeurs brutes du noyau ; `ENRICHED` ajoute notamment des informations interprétées comme les UID ou GID. |
| `name_format` | `NONE`, `HOSTNAME`, `FQD`, `NUMERIC`, `USER` | Format du nom d’hôte ajouté aux événements. `USER` permet une valeur définie par `name`. |
| `name` | chaîne | Nom utilisé lorsque `name_format = USER`. |
| `log_group` | groupe Unix | Groupe propriétaire ou groupe associé aux journaux selon les permissions gérées par la distribution. |
| `flush` | `none`, `incremental`, `incremental_async`, `data`, `sync` | Politique d’écriture. `sync` privilégie la durabilité mais peut augmenter le coût d’I/O ; `incremental_async` offre généralement un compromis. |
| `freq` | entier positif | Nombre d’enregistrements avant demande de vidage lorsque `flush = incremental` ou `incremental_async`. |

### Rotation et conservation

| Clé | Valeurs attendues | Usage |
|---|---|---|
| `max_log_file` | entier positif en MiB | Taille maximale d’un fichier de journal avant rotation. |
| `num_logs` | entier positif | Nombre de fichiers conservés pendant la rotation. |
| `max_log_file_action` | `ignore`, `syslog`, `suspend`, `rotate`, `keep_logs`, `exec` | Action lorsque `max_log_file` est atteint. `keep_logs` évite la suppression automatique des anciens fichiers ; `exec` lance le programme défini par `space_left_action`? Non : utilisez la valeur/documentation de votre version, car les actions disponibles varient. |
| `delete` | `yes` ou `no` | Autorise ou non la suppression des anciens fichiers lors de la rotation, selon la version d’audit. |

> Pour les environnements réglementés, dimensionnez `num_logs`, contrôlez l’espace disque et préférez une stratégie qui évite l’écrasement silencieux. La rotation externe par `logrotate` doit être coordonnée avec `auditd` et ne doit pas supprimer les journaux requis par la politique de conservation.

### Réaction à la saturation

| Clé | Valeurs attendues | Usage |
|---|---|---|
| `space_left` | entier avec unité (`M`, `G`, etc.) ou pourcentage selon version | Seuil d’espace libre déclenchant `space_left_action`. |
| `space_left_action` | `ignore`, `syslog`, `rotate`, `email`, `exec`, `suspend`, `single`, `halt` | Action au franchissement du seuil d’espace libre. `email` nécessite une configuration adaptée ; `halt` et `single` sont des mesures très disruptives. |
| `admin_space_left` | entier avec unité ou pourcentage | Seuil critique réservé à l’administrateur. Il doit être inférieur à `space_left`. |
| `admin_space_left_action` | mêmes familles d’actions | Réaction au franchissement du seuil critique. |
| `disk_full_action` | `ignore`, `syslog`, `exec`, `suspend`, `single`, `halt` | Réaction lorsque le système de fichiers ne dispose plus d’espace. |
| `disk_error_action` | `ignore`, `syslog`, `exec`, `suspend`, `single`, `halt` | Réaction à une erreur d’écriture sur le système de fichiers. |

### Intégrité et comportement du démon

| Clé | Valeurs attendues | Usage |
|---|---|---|
| `priority_boost` | entier, généralement `0` à `20` | Ajustement de priorité du démon. Une valeur excessive peut pénaliser d’autres services. |
| `admin_space_left_action` | action documentée ci-dessus | Protection de dernier niveau avant épuisement du stockage. |
| `verify_email` | `yes` ou `no` | Vérifie l’adresse utilisée pour les notifications selon la version. |
| `action_mail_acct` | adresse e-mail ou compte | Destinataire des notifications lorsque l’action `email` est utilisée. |
| `krb5_principal` | principal Kerberos | Identité utilisée pour l’envoi via une configuration Kerberos, si prise en charge. |
| `krb5_secret` | chemin de fichier | Secret Kerberos ; protéger strictement les permissions si cette option est utilisée. |

Les clés disponibles varient selon la version de `auditd`. Consultez toujours `man auditd.conf` installé sur la machine et la documentation du paquet (`audit` ou `auditd`).

## Exemple prudent

```ini
log_file = /var/log/audit/audit.log
log_group = adm
log_format = ENRICHED
flush = incremental_async
freq = 50
max_log_file = 50
num_logs = 10
max_log_file_action = rotate
space_left = 1G
space_left_action = syslog
admin_space_left = 500M
admin_space_left_action = single
disk_full_action = suspend
disk_error_action = suspend
name_format = HOSTNAME
```

Adaptez les seuils à la taille du système de fichiers et à la politique de conservation. Les actions `single` et `halt` peuvent interrompre la disponibilité du système : elles doivent être testées et validées avant production.

## Appliquer et vérifier

Après modification, validez la syntaxe et rechargez le service selon la distribution :

```bash
sudo systemctl restart auditd
sudo systemctl status auditd --no-pager
sudo journalctl -u auditd -b
```

Sur certaines distributions, `auditd` ne peut pas être redémarré avec `systemctl restart` de manière classique ; utilisez alors le mécanisme fourni par le paquet, par exemple :

```bash
sudo service auditd restart
```

Vérifiez ensuite :

```bash
sudo auditctl -s
sudo ausearch -m DAEMON_START,DAEMON_CONFIG -ts recent
```

## Points d’attention

- Protégez `/etc/audit/auditd.conf` et `/var/log/audit` contre les modifications non autorisées.
- Assurez-vous que le stockage des journaux est local, dimensionné et surveillé.
- Évitez de placer les journaux sur un système de fichiers susceptible d’être saturé par une application non maîtrisée.
- Documentez les actions de panne : `suspend`, `single` et `halt` ont un impact opérationnel important.
- Ne confondez pas `auditd.conf` avec les règles : les règles persistantes sont généralement dans `/etc/audit/rules.d/` et sont chargées par `augenrules`.

## Références

- [`auditd.conf(5)` — man7.org](https://man7.org/linux/man-pages/man5/auditd.conf.5.html)
- [`auditd(8)` — man7.org](https://man7.org/linux/man-pages/man8/auditd.8.html)
- [`auditctl(8)` — man7.org](https://man7.org/linux/man-pages/man8/auditctl.8.html)
- [Linux Audit documentation](https://github.com/linux-audit/audit-documentation)
