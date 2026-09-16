# Outils CLI auditd

Les commandes suivantes nécessitent souvent `root` ou `sudo`. Les options peuvent varier selon la version du paquet `audit`.

## `auditctl` — gérer les règles actives

```bash
sudo auditctl -s                 # état du sous-système
sudo auditctl -l                 # règles actuellement chargées
sudo auditctl -w /etc/passwd -p wa -k identity
sudo auditctl -a always,exit -F arch=b64 -S openat -F dir=/etc -F auid>=1000 -F auid!=unset -k etc_access
sudo auditctl -d always,exit -F arch=b64 -S openat -F dir=/etc -F auid>=1000 -F auid!=unset -k etc_access
sudo auditctl -D                 # supprimer toutes les règles actives
```

Options importantes :

- `-l` affiche les règles chargées ; `-D` les supprime.
- `-w PATH` surveille un fichier ou répertoire (`-p r|w|x|a`, `-k MOTCLE`). Cette forme est simple mais moins performante que les règles syscall modernes.
- `-a LIST,ACTION` ajoute une règle : `LIST` vaut notamment `always,exit` et `ACTION` vaut `always` ou `never`.
- `-F` ajoute un filtre : `arch=b64`, `uid=`, `auid=`, `exe=`, `dir=`, `success=`, etc.
- `-S` sélectionne un syscall ; utiliser plusieurs `-S` avec prudence et vérifier l’architecture.
- `-b` règle le buffer noyau, `-f` le comportement en cas d’échec (`0`, `1`, `2`).

Les modifications avec `auditctl` sont temporaires : elles ne survivent pas au redémarrage. Utiliser `/etc/audit/rules.d/*.rules` puis `augenrules --load` pour la persistance.

## `augenrules` — compiler et charger les règles

```bash
sudo augenrules --check       # vérifier la génération
sudo augenrules --load        # générer /etc/audit/audit.rules et charger
sudo augenrules --debug       # afficher les détails de traitement
```

Les fichiers `*.rules` sont lus dans l’ordre lexical. Éviter d’éditer directement `/etc/audit/audit.rules` lorsqu’`augenrules` est utilisé.

## `ausearch` — rechercher les événements

```bash
sudo ausearch -m USER_LOGIN -ts today
sudo ausearch -k identity -i
sudo ausearch -ua 1001 -ts recent
sudo ausearch -f /etc/passwd -ts 09/16/2026 09:00:00
sudo ausearch -sc execve --success no -i
sudo ausearch -a 1234 -i
sudo ausearch --input-logs --format text
```

Options utiles :

- `-m TYPE` filtre le type d’événement ; plusieurs types peuvent être donnés.
- `-k KEY` recherche la clé d’une règle ; `-f PATH` recherche un chemin.
- `-ts` et `-te` définissent début et fin (`today`, `yesterday`, `recent`, ou date/heure).
- `-ua UID`, `-ui UID`, `-ul UID` filtrent les identités réelle, effective ou de connexion.
- `-sc SYSCALL`, `-a EVENT_ID`, `--success yes|no` affinent la recherche.
- `-i` interprète les valeurs numériques ; `--raw` conserve le format brut.
- `-x` affiche l’exécutable, `--just-one` limite le résultat.

Pour reconstruire un événement multi-lignes, conserver les enregistrements partageant le même `msg=audit(...:ID)`.

## `aureport` — produire des synthèses

```bash
sudo aureport
sudo aureport -au -ts today       # authentifications
sudo aureport -l -i               # connexions
sudo aureport -f -i               # accès fichiers
sudo aureport -x --summary        # exécutables
sudo aureport -s --failed         # syscalls échoués
sudo aureport -t                  # période couverte
```

`-au`, `-l`, `-f`, `-x` et `-s` sélectionnent des rapports ; `-ts`/`-te` filtrent la période ; `-i` interprète les identifiants ; `--failed` limite aux échecs ; `--summary` agrège les résultats.

## `auditd`

```bash
sudo systemctl status auditd
sudo systemctl restart auditd
sudo auditd -f                  # mode premier plan, diagnostic
sudo auditd -s enable           # activer l’audit selon la version
```

La gestion normale passe par systemd et la configuration `/etc/audit/auditd.conf`. Vérifier la syntaxe et les capacités disponibles sur la distribution avant toute modification.

## `audisp-remote` et plugins

```bash
sudo systemctl status auditd
sudo ls -l /etc/audit/plugins.d/
sudo ausearch --input-logs -ts today
```

Les plugins sont configurés dans `/etc/audit/plugins.d/`. `audisp-remote` permet l’envoi vers un collecteur configuré dans `audisp-remote.conf`; protéger le transport et tester la perte de connectivité.

## Manpages et références

```bash
man auditctl
man augenrules
man ausearch
man aureport
man auditd
man audit.rules
man auditd.conf
man audisp-remote
```

- [Linux Audit documentation](https://github.com/linux-audit/audit-documentation)
- [Projet Linux Audit](https://github.com/linux-audit)
- [Manpages Debian — auditctl](https://manpages.debian.org/bookworm/auditd/auditctl.8.en.html)
- [Manpages Debian — ausearch](https://manpages.debian.org/bookworm/auditd/ausearch.8.en.html)
- [Manpages Debian — aureport](https://manpages.debian.org/bookworm/auditd/aureport.8.en.html)
