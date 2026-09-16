# AppArmor — administration et analyse

```bash
sudo aa-status
sudo aa-enabled
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
sudo systemctl reload apparmor
```

## Génération assistée

`aa-genprof` observe une application et propose un profil ; `aa-logprof` analyse les refus existants. Ces outils accélèrent le démarrage mais chaque proposition doit être validée selon le besoin métier.

## Analyse

```bash
sudo journalctl -k | grep -i 'apparmor="DENIED"'
sudo dmesg | grep -i apparmor
```

Identifier profil, opération, chemin, processus et décision. Corriger le profil ou l'application, puis recharger et retester.