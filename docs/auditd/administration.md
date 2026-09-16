# auditd — administration et analyse

```bash
sudo auditctl -s
sudo auditctl -l
sudo ausearch -k identity -i
sudo ausearch -m USER_LOGIN -ts today -i
sudo aureport --summary
sudo aureport --auth --success
sudo systemctl status auditd
```

## Lecture d'un événement

Comparer `auid` (identité de connexion), `uid` (identité au moment de l'action), `euid`, `pid`, `ppid`, `exe`, `syscall`, `success`, `exit`, `name`, `cwd` et `key`. Un événement peut être composé de plusieurs enregistrements partageant le même identifiant.

## Actions

Les règles ajoutées avec `auditctl` sont temporaires. Pour les conserver, les placer dans `rules.d`. Une règle de verrouillage (`-e 2`) empêche les changements jusqu'au redémarrage : ne l'utiliser qu'après validation complète.