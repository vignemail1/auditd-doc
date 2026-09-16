# Référence rapide des commandes

Les options exactes peuvent différer selon la distribution et la version. Consulter d’abord `commande --help`, puis la manpage locale.

## auditd

| Commande | Usage | Exemple |
|---|---|---|
| `auditctl` | Règles temporaires et état | `sudo auditctl -s`, `sudo auditctl -l` |
| `augenrules` | Charger les règles persistantes | `sudo augenrules --load` |
| `ausearch` | Rechercher les événements | `sudo ausearch -k identity -i` |
| `aureport` | Rapports synthétiques | `sudo aureport -au -i` |
| `systemctl` | Service auditd | `sudo systemctl status auditd` |

## AppArmor

| Commande | Usage | Exemple |
|---|---|---|
| `aa-status` | État et profils | `sudo aa-status` |
| `aa-enforce` | Mode bloquant | `sudo aa-enforce profile` |
| `aa-complain` | Journaliser sans bloquer | `sudo aa-complain profile` |
| `aa-logprof` | Proposer des règles | `sudo aa-logprof` |
| `aa-genprof` | Générer un profil | `sudo aa-genprof /usr/sbin/app` |
| `apparmor_parser` | Charger/retirer un profil | `sudo apparmor_parser -r profile` |
| `aa-exec` | Exécuter sous profil | `sudo aa-exec -p profile -- commande` |

## SELinux

| Commande | Usage | Exemple |
|---|---|---|
| `getenforce` / `setenforce` | Lire/modifier le mode | `getenforce`, `sudo setenforce 1` |
| `sestatus` | État détaillé | `sestatus -b -v` |
| `semanage` | Personnalisations persistantes | `sudo semanage fcontext -l` |
| `restorecon` | Appliquer les contextes | `sudo restorecon -RFv /srv/www` |
| `ausearch` | Chercher les AVC | `sudo ausearch -m AVC -ts recent -i` |
| `audit2why` | Expliquer un refus | `sudo ausearch -m AVC --raw \| audit2why` |
| `audit2allow` | Proposer un module | `sudo ausearch -m AVC --raw \| audit2allow -M local` |
| `sesearch` / `seinfo` | Interroger la politique | `sesearch -A -s httpd_t` |
| `semodule` | Gérer les modules | `sudo semodule -l` |

## Obtenir de l’aide

```bash
commande --help
man commande
man 5 audit.rules
man 5 auditd.conf
apropos audit
apropos apparmor
apropos selinux
```

Pour les commandes qui lisent des journaux sensibles, conserver les droits d’accès et éviter de copier des sorties contenant des identifiants, chemins internes ou données personnelles dans un ticket public.
