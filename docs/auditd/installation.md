# auditd — installation et architecture

## Installation

Debian/Ubuntu :

```bash
sudo apt update && sudo apt install auditd audispd-plugins
```

RHEL/Fedora :

```bash
sudo dnf install audit audit-libs audispd-plugins
```

Vérifier :

```bash
sudo systemctl enable --now auditd
sudo auditctl -s
sudo journalctl -u auditd
```

Le nom du paquet et les possibilités de redémarrage peuvent varier. Le noyau doit exposer le sous-système audit et l'horloge doit être fiable.

## Architecture

Le noyau produit des événements ; `auditd` les écrit selon `/etc/audit/auditd.conf`. `auditctl` manipule les règles actives. `augenrules` assemble les fichiers de `/etc/audit/rules.d/`. `ausearch` et `aureport` exploitent les journaux.