# Outils CLI AppArmor

La plupart des commandes proviennent de `apparmor-utils`. Les profils sont généralement dans `/etc/apparmor.d/`.

## `aa-status` et `aa-enabled`

```bash
sudo aa-status
sudo aa-status --enabled
sudo aa-enabled; echo $?
sudo aa-status --profiled
```

`aa-status` affiche l’état du module, les profils chargés et leurs modes (`enforce` ou `complain`). `aa-enabled` retourne un code non nul si AppArmor n’est pas actif.

## `aa-enforce`, `aa-complain`, `aa-disable`

```bash
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx
sudo aa-enforce usr.sbin.nginx
```

- `aa-enforce` applique les refus du profil.
- `aa-complain` journalise les violations sans les bloquer.
- `aa-disable` décharge/désactive le profil ; à réserver aux tests ou diagnostics.
- Plusieurs profils peuvent être fournis dans une même commande.

Après modification, recharger le profil :

```bash
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
sudo systemctl reload apparmor
```

## `aa-complain` et génération de profils

```bash
sudo aa-genprof /usr/sbin/mon-service
sudo aa-logprof
sudo aa-logprof -f /var/log/audit/audit.log
sudo aa-cleanprof /etc/apparmor.d/usr.sbin.mon-service
```

`aa-genprof` guide la création d’un profil à partir de l’activité observée. `aa-logprof` propose des règles à partir des refus journalisés. Toujours relire les propositions : ne pas autoriser automatiquement des chemins ou capacités non nécessaires.

## `apparmor_parser`

```bash
sudo apparmor_parser -p /etc/apparmor.d/usr.sbin.nginx  # prévisualiser
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx  # remplacer
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.nginx  # retirer
sudo apparmor_parser -T /etc/apparmor.d/usr.sbin.nginx  # afficher les stats
sudo apparmor_parser -Q /etc/apparmor.d/usr.sbin.nginx  # mode silencieux selon version
```

Selon la version, `-r` remplace un profil chargé, `-R` le retire et `-p` prévisualise le profil compilé. Utiliser `apparmor_parser --help` et la manpage locale pour les options exactes.

## `aa-exec`

```bash
sudo aa-exec -p usr.sbin.nginx -- nginx -t
sudo aa-exec -p unconfined -- id
sudo aa-exec -d /usr/sbin/mon-service -- /usr/sbin/mon-service
```

`-p` sélectionne un profil, `-d` demande un changement de répertoire de profil selon les versions, et `--` sépare les options d’`aa-exec` de la commande.

## Diagnostic et journaux

```bash
sudo journalctl -k -g 'apparmor="DENIED"'
sudo journalctl -u apparmor
sudo dmesg | grep -i apparmor
sudo aa-notify -s 1 -v
sudo aa-notify -p -f /var/log/audit/audit.log
```

`aa-notify` résume les refus récents ; `-s` indique une période en secondes, `-p` affiche les profils concernés, et `-f` choisit un fichier d’audit lorsque disponible.

## Manpages et références

```bash
man aa-status
man aa-enforce
man aa-complain
man aa-logprof
man aa-genprof
man apparmor_parser
man aa-exec
man apparmor.d
```

- [Documentation officielle Ubuntu AppArmor](https://ubuntu.com/server/docs/security-apparmor)
- [Projet AppArmor](https://apparmor.net/)
- [Documentation openSUSE AppArmor](https://documentation.suse.com/sles/)
- [Manpages Ubuntu AppArmor](https://manpages.ubuntu.com/manpages/jammy/man8/)
- [Référentiel AppArmor](https://gitlab.com/apparmor/apparmor)
