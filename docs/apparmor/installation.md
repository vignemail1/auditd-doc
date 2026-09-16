# AppArmor — installation et architecture

Debian/Ubuntu :

```bash
sudo apt install apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra
sudo systemctl enable --now apparmor
sudo aa-status
```

Sur les autres distributions, vérifier que le noyau possède AppArmor, que le paramètre de démarrage l'active et que les paquets correspondent à la version. Un redémarrage peut être requis après activation du support noyau.

## Répertoires

- `/etc/apparmor.d/` : profils ;
- `/etc/apparmor.d/tunables/` : variables et chemins communs ;
- `/var/lib/apparmor/` : état généré selon distribution ;
- journaux noyau/journald : événements de refus.

```bash
sudo aa-status
sudo apparmor_parser -r /etc/apparmor.d/mon-profil
```