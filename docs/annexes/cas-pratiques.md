# Cas pratiques

## Surveiller un fichier sensible avec auditd

```bash
sudo auditctl -w /etc/ssh/sshd_config -p wa -k ssh-config
sudo ausearch -k ssh-config -i
```

Pour la persistance, placer une règle équivalente dans `/etc/audit/rules.d/` puis charger avec `augenrules --load`.

## Mettre AppArmor en observation

```bash
sudo aa-complain /etc/apparmor.d/usr.sbin.mysqld
sudo journalctl -k | grep -i apparmor
sudo aa-logprof
sudo aa-enforce /etc/apparmor.d/usr.sbin.mysqld
```

## Diagnostiquer un refus SELinux

```bash
sudo ausearch -m AVC -ts recent
sudo sealert -a /var/log/audit/audit.log
```

Corriger la cause — contexte, booléen, port ou politique — plutôt que générer automatiquement une autorisation aveugle.

## Protéger un service

1. établir ses fichiers, ports et capacités nécessaires ;
2. observer l'exécution nominale ;
3. limiter les accès ;
4. tester les opérations d'administration ;
5. surveiller après déploiement.