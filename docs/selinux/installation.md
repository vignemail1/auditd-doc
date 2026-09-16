# SELinux — installation et architecture

Sur Fedora/RHEL :

```bash
sudo dnf install policycoreutils policycoreutils-python-utils selinux-policy-targeted setroubleshoot-server
getenforce
sestatus
```

Sur Debian/Ubuntu, les paquets et le support varient ; vérifier la documentation de la version avant migration. Changer de politique ou activer SELinux sur un système existant peut nécessiter un réétiquetage et une procédure de démarrage de secours.

## Modes

- `Enforcing` : décisions appliquées ;
- `Permissive` : refus journalisés mais non bloquants ;
- `Disabled` : mécanisme désactivé, déconseillé.

```bash
sudo setenforce 1
sudo getenforce
```

La configuration persistante se trouve généralement dans `/etc/selinux/config`.