# auditd — configuration et règles

## `auditd.conf`

Paramètres importants : `log_file`, `log_group`, `log_format`, `flush`, `freq`, `max_log_file`, `num_logs`, `max_log_file_action`, `space_left_action`, `admin_space_left_action`, `disk_full_action` et `disk_error_action`.

Définir une stratégie explicite : alerter avant saturation, conserver assez de rotations et éviter une action qui interromprait un serveur sans procédure validée.

## Règles persistantes

Créer par exemple `/etc/audit/rules.d/50-local.rules` :

```text
-w /etc/ssh/sshd_config -p wa -k ssh-config
-w /etc/passwd -p wa -k identity
-a always,exit -F arch=b64 -S setuid,setgid -k identity-change
```

Charger et vérifier :

```bash
sudo augenrules --load
sudo auditctl -l
```

## Principes de conception

Préférer une clé (`-k`) par objectif. Filtrer par `auid` ou `uid` avec prudence. Sur les systèmes 64 bits, évaluer les architectures pertinentes. Les règles `-w` sont simples mais moins fines que les règles syscall. Tester le volume avant généralisation.