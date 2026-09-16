# auditd — outils complémentaires

- `systemd-journald` : événements système, complément et non remplacement de l'audit ;
- `rsyslog` ou `syslog-ng` : routage ;
- `audisp-remote` : transmission audit ;
- `ausearch` et `aureport` : investigation ;
- `jq`, scripts Python et SIEM : normalisation et corrélation ;
- `tuned`, `iostat`, `sar` : mesure d'impact ;
- outils de conformité CIS/STIG : contrôle de couverture.

Limiter les collecteurs et scripts aux données nécessaires. Tester leur comportement lorsque le réseau ou le stockage distant est indisponible.