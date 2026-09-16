# Construire des règles auditd

Cette page explique comment traduire un besoin de traçabilité en règle `auditctl` persistante. Les règles sont évaluées par le noyau Linux Audit ; `auditd` collecte ensuite les événements et les écrit dans le journal configuré.

> **Prudence :** tester les règles sur une machine représentative. Une règle trop large peut générer beaucoup d'événements, consommer de l'espace disque et dégrader les performances.

## Où placer les règles

- `/etc/audit/rules.d/*.rules` : fichiers persistants, chargés par `augenrules`.
- `/etc/audit/audit.rules` : fichier généré ou chargé directement selon la distribution.
- `auditctl ...` : modification immédiate, généralement non persistante.

Après modification :

```bash
sudo augenrules --check
sudo augenrules --load
sudo auditctl -l
```

Selon la distribution, un redémarrage de `auditd` peut être nécessaire pour la collecte, mais le chargement des règles se fait normalement avec `augenrules --load`.

## Structure générale

Une règle peut sélectionner :

1. une architecture et une famille d'appels système ;
2. un ou plusieurs appels système ;
3. des filtres (`uid`, `auid`, chemin, résultat, etc.) ;
4. une clé de recherche (`-k`) ;
5. une action (`-a`) ou une surveillance de chemin (`-w`).

Exemple :

```text
-a always,exit -F arch=b64 -S unlink,unlinkat -F auid>=1000 -F auid!=-1 -k suppression_fichiers
```

Cette règle journalise les suppressions effectuées par un utilisateur authentifié normal, sur une machine 64 bits.

## Options de règles et valeurs attendues

### `-w chemin`

Surveille un fichier ou un répertoire :

```bash
sudo auditctl -w /etc/ssh/sshd_config -p wa -k ssh_config
```

- **Valeur attendue :** un chemin absolu existant ou destiné à être surveillé.
- Avec un fichier : événements concernant ce fichier.
- Avec un répertoire : la surveillance est récursive selon les capacités et la version du noyau ; pour des règles modernes et précises, préférer `-a ... -F path=` ou `-F dir=`.
- `-w` est simple mais ancien et moins flexible que les règles syscall.

### `-p permissions`

Précise les types d'accès surveillés avec `-w` :

| Valeur | Signification |
|---|---|
| `r` | lecture du fichier |
| `w` | écriture/modification du contenu |
| `x` | exécution |
| `a` | modification des attributs : propriétaire, permissions, timestamps, ACL, etc. |

Les lettres peuvent être combinées : `-p wa`, `-p rwxa`. La valeur `-p` n'est pas une permission Unix octale : elle décrit les accès à auditer.

```bash
sudo auditctl -w /etc/passwd -p wa -k identite
```

### `-a action,list`

Ajoute une règle à une liste du noyau :

```text
-a always,exit -F arch=b64 -S openat -F dir=/etc -k lecture_etc
```

- **`action`** : généralement `always` pour générer un événement ou `never` pour exclure un événement.
- **`list`** : le plus souvent `exit` pour filtrer le résultat d'un appel système ; autres listes selon le noyau : `task`, `user`, `exclude`, `filesystem`, `io_uring`.

Pour les règles d'appels système, utiliser en général `always,exit`. L'action `never` doit être utilisée avec une grande prudence car elle peut supprimer des événements attendus.

### `-F field=value`

Ajoute un filtre. Les champs courants sont :

| Filtre | Valeur attendue et usage |
|---|---|
| `arch=b32` / `arch=b64` | ABI 32 ou 64 bits ; préciser les deux si des programmes 32 bits peuvent être utilisés |
| `auid=UID` | identifiant de l'utilisateur authentifié, conservé après `sudo` |
| `auid>=1000` | utilisateurs humains selon la convention locale |
| `auid!=-1` | exclut les sessions sans identifiant valide ; écrire aussi `unset` selon les outils/version |
| `uid=UID` | UID effectif au moment de l'appel |
| `euid=UID` | UID effectif |
| `suid=UID` | UID sauvegardé |
| `fsuid=UID` | UID utilisé pour les contrôles du système de fichiers |
| `gid`, `egid`, `sgid`, `fsgid` | équivalents pour les groupes |
| `path=/chemin` | fichier précis ; chemin absolu |
| `dir=/répertoire` | répertoire à surveiller, principalement avec les règles syscall |
| `success=1` / `success=0` | appel réussi ou échoué |
| `exit=CODE` | code de retour exact ; souvent une valeur errno numérique |
| `perm=r|w|x|a` | type d'accès, utile avec `path`/`dir` selon le support |
| `exe=/chemin/binaire` | exécutable exact à l'origine de l'appel |
| `msgtype=TYPE` | type d'événement, surtout dans les listes d'exclusion |
| `key=texte` | clé de recherche ; `-k texte` est la forme abrégée |

Les comparateurs généralement disponibles sont `=`, `!=`, `<`, `<=`, `>`, `>=`. Les valeurs numériques doivent être adaptées aux UID/GID et aux codes errno du système.

Exemples :

```text
-F auid>=1000 -F auid!=-1
-F uid=0
-F path=/etc/sudoers
-F success=0
-F exe=/usr/bin/passwd
```

### `-S syscall`

Indique un appel système à surveiller :

```text
-S execve
-S openat
-S setuid,setreuid,setresuid
```

- **Valeur attendue :** nom d'un syscall reconnu par l'architecture ciblée.
- Plusieurs syscalls peuvent être séparés par des virgules.
- Ne pas mélanger des syscalls propres à des architectures différentes dans une règle sans vérifier le résultat.

Appels fréquemment utilisés :

| Besoin | Syscalls fréquents |
|---|---|
| exécution | `execve`, `execveat` |
| ouverture/lecture/écriture | `open`, `openat`, `open_by_handle_at` |
| suppression | `unlink`, `unlinkat`, `rename`, `renameat`, `renameat2` |
| permissions et propriété | `chmod`, `fchmod`, `fchmodat`, `chown`, `fchownat` |
| identités privilégiées | `setuid`, `setgid`, `setresuid`, `setresgid`, `capset` |
| modules noyau | `init_module`, `finit_module`, `delete_module` |

Pour connaître les syscalls disponibles :

```bash
man 2 syscalls
ausyscall --dump | less
ausyscall x86_64 openat
```

### `-k clé` et `-F key=clé`

Associe une chaîne de recherche à la règle :

```text
-a always,exit -F arch=b64 -S execve -F auid>=1000 -F auid!=-1 -k executions_utilisateur
```

- **Valeur attendue :** texte court, explicite et stable ; éviter les espaces.
- Recherche : `ausearch -k executions_utilisateur -i`.
- Une règle peut avoir une seule clé pratique pour l'exploitation et le tri des alertes.

### `-e valeur`

Contrôle l'état de l'audit :

```text
-e 1
```

Valeurs :

| Valeur | Effet |
|---|---|
| `0` | audit désactivé |
| `1` | audit activé |
| `2` | configuration verrouillée jusqu'au redémarrage |

Placer `-e 2` en dernière ligne du jeu de règles, après toutes les règles. Une fois verrouillé, le noyau refuse les modifications jusqu'au redémarrage. Tester d'abord avec `-e 1`.

### `-b backlog`

Définit la taille de la file d'attente noyau :

```text
-b 8192
```

- **Valeur attendue :** entier positif.
- Une valeur trop faible peut provoquer des pertes en cas de pointe ; une valeur trop élevée consomme davantage de mémoire.

### `--backlog_wait_time millisecondes`

Temps pendant lequel le noyau attend lorsqu'il ne peut pas remettre immédiatement un événement :

```text
--backlog_wait_time 60000
```

- **Valeur attendue :** entier en millisecondes.
- À calibrer avec `-b`, la charge et les exigences de non-perte.

### `-f mode`

Action lorsque le noyau rencontre une condition critique ou ne peut pas journaliser :

| Mode | Comportement |
|---|---|
| `0` | ignorer l'erreur |
| `1` | écrire un avertissement dans les logs |
| `2` | déclencher une panique noyau |

```text
-f 1
```

Le mode `2` est réservé aux environnements dont la continuité de sécurité exige l'arrêt plutôt que l'exécution sans audit.

### `-D`

Supprime toutes les règles actuellement chargées :

```text
-D
```

À placer au début d'un fichier de règles pour éviter les doublons lors d'un rechargement. Ne jamais l'exécuter sans recharger immédiatement un jeu de règles valide.

### `-c mode`

Ancien contrôle de priorité :

| Mode | Sens |
|---|---|
| `0` | aucune garantie de priorité |
| `1` | priorité basse |
| `2` | priorité haute |

Les noyaux et versions récentes privilégient les paramètres de backlog et `--backlog_wait_time`. Vérifier `man auditctl` avant utilisation.

## Méthode de conception

### 1. Définir le besoin

Exemples : « savoir qui modifie `/etc/ssh/sshd_config` », « savoir quels utilisateurs lancent des commandes privilégiées », « détecter les suppressions de fichiers ».

### 2. Choisir le mécanisme

- Fichier ou répertoire précis : `-a always,exit ... -F path/dir=...`.
- Accès générique par syscall : `-S` avec `arch` et filtres.
- Règle simple de compatibilité : `-w`, `-p`.

### 3. Réduire le périmètre

Toujours envisager `arch`, `auid`, `uid`, `exe`, `path`, `dir`, `success`. Une règle limitée est plus exploitable qu'une règle globale.

### 4. Ajouter une clé

Utiliser une convention, par exemple `identite_*`, `privilege_*`, `config_*`, `execution_*`.

### 5. Tester et mesurer

```bash
sudo auditctl -D
sudo auditctl -a always,exit -F arch=b64 -S execve -F auid>=1000 -F auid!=-1 -k test_exec
sudo ausearch -k test_exec -ts recent -i
sudo auditctl -s
```

Ne supprimez pas les règles de production avec `-D` sans procédure de restauration ; pour un test réel, charger un fichier contrôlé avec `augenrules` dans une fenêtre prévue.

## Exemples prêts à adapter

### Modifications de fichiers sensibles

```text
-a always,exit -F arch=b64 -F path=/etc/passwd -F perm=wa -F auid!=-1 -k identite
-a always,exit -F arch=b64 -F path=/etc/shadow -F perm=wa -F auid!=-1 -k identite
-a always,exit -F arch=b64 -F path=/etc/sudoers -F perm=wa -F auid!=-1 -k privilege_config
```

Ajouter les règles `arch=b32` si la plateforme autorise effectivement des binaires 32 bits.

### Exécutions d'utilisateurs connectés

```text
-a always,exit -F arch=b64 -S execve,execveat -F auid>=1000 -F auid!=-1 -k execution_utilisateur
```

Cette règle peut être volumineuse sur un serveur très actif ; filtrer éventuellement par `exe` ou utiliser une stratégie de collecte adaptée.

### Suppressions et renommages

```text
-a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=1000 -F auid!=-1 -k suppression_renommage
```

### Chargement de modules noyau

```text
-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -F auid!=-1 -k modules_noyau
```

## Vérification et recherche

```bash
sudo auditctl -l
sudo auditctl -s
sudo ausearch -k identite -i
sudo ausearch -sc execve -ts today -i
sudo aureport --summary
```

La sortie `ausearch -i` convertit les UID, architectures et autres valeurs en représentations lisibles.

## Références officielles et manpages

- [Linux Audit — documentation](https://github.com/linux-audit/audit-documentation)
- [audit-userspace](https://github.com/linux-audit/audit-userspace)
- [auditctl(8), man7.org](https://man7.org/linux/man-pages/man8/auditctl.8.html)
- [audit.rules(7), man7.org](https://man7.org/linux/man-pages/man7/audit.rules.7.html)
- [ausearch(8), man7.org](https://man7.org/linux/man-pages/man8/ausearch.8.html)
- [ausyscall(8), man7.org](https://man7.org/linux/man-pages/man8/ausyscall.8.html)
- [auditd.conf(5), man7.org](https://man7.org/linux/man-pages/man5/auditd.conf.5.html)

Manpages locales utiles :

```bash
man 8 auditctl
man 7 audit.rules
man 8 ausearch
man 8 ausyscall
man 5 auditd.conf
```
