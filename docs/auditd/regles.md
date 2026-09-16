# Règles Auditd

Cette page décrit les règles qui déterminent **quoi** le noyau doit enregistrer. La configuration du démon et des journaux est documentée dans [Configuration d’auditd](configuration.md).

## Emplacements et chargement

Les règles temporaires sont chargées avec `auditctl` et disparaissent au redémarrage :

```bash
sudo auditctl -R /chemin/vers/regles.rules
sudo auditctl -l
```

Pour les règles persistantes, utilisez généralement :

```text
/etc/audit/rules.d/*.rules
```

Puis compilez et chargez-les :

```bash
sudo augenrules --check
sudo augenrules --load
sudo auditctl -l
```

Le fichier généré est souvent `/etc/audit/audit.rules`. Évitez de modifier directement ce fichier lorsqu’il est géré par `augenrules`.

## Règles de contrôle

### Verrouiller la configuration

```text
-e 2
```

Valeurs courantes de `-e` :

- `0` : désactive l’audit ;
- `1` : active l’audit ;
- `2` : active l’audit et verrouille la configuration jusqu’au redémarrage.

Avec `-e 2`, toute modification ultérieure des règles est refusée. Placez cette directive en dernier et testez soigneusement les règles avant de l’activer.

### Nombre de messages noyau

```text
-b 8192
```

`-b` définit la taille du tampon noyau en nombre de messages. Augmentez-la si vous observez des pertes sous forte charge, tout en surveillant la mémoire.

### Réaction aux pertes

```text
-f 1
```

`-f` définit l’action en cas d’erreur critique ou de perte de données :

- `0` : `silent`, aucune alerte ;
- `1` : `printk`, message noyau ;
- `2` : `panic`, panique du noyau.

Le choix dépend de la politique de disponibilité et d’intégrité. `2` est très strict et doit être validé en exploitation.

## Règles de surveillance de chemins : `-w`

Syntaxe :

```text
-w /chemin -p rwxa -k identifiant
```

`-w` ajoute une surveillance d’un fichier ou d’un répertoire.

- chemin absolu attendu ;
- avec un répertoire, le comportement exact dépend de la version et de la règle : pour une couverture fine, préférez les règles syscall avec filtres ;
- `-k` ajoute une clé de recherche facultative ;
- `-p` sélectionne les permissions surveillées.

Permissions de `-p` :

| Lettre | Signification |
|---|---|
| `r` | lecture du contenu |
| `w` | écriture ou modification |
| `x` | exécution |
| `a` | modification des attributs : permissions, propriétaire, horodatage, ACL, etc. |

Exemple :

```text
-w /etc/ssh/sshd_config -p wa -k ssh_config
```

Cette règle surveille les modifications et changements d’attributs, mais pas chaque lecture du fichier.

> Les règles `-w` sont historiques et moins expressives. Les règles syscall `-a always,exit -F path=...` sont préférables lorsqu’une granularité et des performances prévisibles sont nécessaires.

## Règles syscall : `-a`

Syntaxe générale :

```text
-a action,filter -S syscall -F champ=valeur -k cle
```

Exemple :

```text
-a always,exit -F arch=b64 -S openat -F dir=/etc -F success=1 -k etc_reads
```

### Action et filtre

Le premier champ de `-a` est l’action :

- `always` : générer un événement ;
- `never` : ne pas générer d’événement.

Le second champ est la liste de filtrage :

- `task` : événements liés à la création de tâches ;
- `user` : événements provenant de l’espace utilisateur ;
- `exit` : filtrage à la sortie d’un appel système ;
- `exclude` : exclure certains types d’événements.

La forme la plus fréquente est `always,exit`.

## Sélectionner les appels système avec `-S`

`-S` indique un appel système. Il accepte un nom ou plusieurs noms séparés par des virgules, selon la syntaxe supportée :

```text
-S openat
-S openat,truncate,rename,unlink
```

Exemples :

```text
-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -k perm_changes
-a always,exit -F arch=b64 -S execve,execveat -F auid>=1000 -k user_exec
```

Utilisez `ausyscall --dump` pour consulter les appels système connus :

```bash
ausyscall --dump
```

## Filtres avec `-F`

`-F` ajoute une condition. La forme est généralement `champ=valeur`, avec des opérateurs de comparaison pour certains champs.

### Architecture

```text
-F arch=b64
-F arch=b32
```

Sur une architecture 64 bits, écrivez souvent une règle pour `b64` et une autre pour `b32` si des programmes 32 bits sont autorisés. Une règle sans architecture peut produire une couverture incomplète ou des résultats inattendus.

### Identités

```text
-F uid=0
-F auid>=1000
-F auid!=unset
-F euid=0
```

- `uid` : UID réel du processus ;
- `euid` : UID effectif ;
- `auid` : UID d’audit associé à la session, généralement conservé après `sudo` ;
- `unset` : valeur non initialisée, souvent représentée par `4294967295`.

Exemple pour les actions effectuées par des utilisateurs connectés :

```text
-a always,exit -F arch=b64 -S execve -F auid>=1000 -F auid!=unset -k user_commands
```

### Chemins et résultats

```text
-F path=/etc/passwd
-F dir=/etc/ssh
-F success=1
-F success=0
```

- `path` cible un fichier ;
- `dir` cible un répertoire pour les appels compatibles ;
- `success=1` sélectionne les succès ;
- `success=0` sélectionne les échecs.

### Comparaisons entre champs

Selon la version :

```text
-F uid!=euid
-F auid!=obj_uid
```

Consultez `auditctl(8)` local pour les champs et opérateurs disponibles.

## Clés de recherche : `-k`

```text
-k ssh_config
```

`-k` associe un identifiant textuel à la règle. Recherchez ensuite les événements avec :

```bash
sudo ausearch -k ssh_config -i
sudo aureport -k --summary
```

Utilisez des clés courtes, stables et explicites. Une même clé peut être utilisée par plusieurs règles formant un même cas d’usage.

## Exemples utiles

### Surveillance des comptes locaux

```text
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k privilege
-w /etc/sudoers.d/ -p wa -k privilege
```

### Exécutions par les utilisateurs

```text
-a always,exit -F arch=b64 -S execve,execveat -F auid>=1000 -F auid!=unset -k user_exec
-a always,exit -F arch=b32 -S execve,execveat -F auid>=1000 -F auid!=unset -k user_exec
```

### Échecs de connexion

Selon le système et les événements générés par PAM :

```text
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins
```

### Changements de permissions

```text
-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -k perm_changes
```

## Tester et dépanner

Afficher les règles actives :

```bash
sudo auditctl -l
sudo auditctl -s
```

Charger une règle temporaire :

```bash
sudo auditctl -w /etc/hosts -p wa -k hosts_test
```

Déclencher un événement, puis rechercher :

```bash
sudo ausearch -k hosts_test -i
sudo ausearch -ts recent -i
sudo aureport --summary
```

Vérifier les erreurs de chargement :

```bash
sudo journalctl -k -b | grep -i audit
sudo journalctl -u auditd -b
sudo augenrules --check
```

## Bonnes pratiques

- Définissez une clé `-k` par cas d’usage.
- Ajoutez `arch=b64` et `arch=b32` lorsque la compatibilité 32 bits est nécessaire.
- Filtrez avec `auid`, `uid`, `path`, `dir` ou `success` pour limiter le volume.
- Évitez de surveiller indistinctement tous les appels système : le volume peut dégrader les performances et saturer les journaux.
- Testez les règles avant `-e 2`.
- Mesurez les pertes et la saturation avec `auditctl -s` et la configuration décrite dans [Configuration d’auditd](configuration.md).

## Références

- [`auditctl(8)` — man7.org](https://man7.org/linux/man-pages/man8/auditctl.8.html)
- [`ausearch(8)` — man7.org](https://man7.org/linux/man-pages/man8/ausearch.8.html)
- [`aureport(8)` — man7.org](https://man7.org/linux/man-pages/man8/aureport.8.html)
- [Linux Audit documentation](https://github.com/linux-audit/audit-documentation)
