# SELinux — contextes et politiques

## Inspection

```bash
ls -Z /var/www/html
ps -eZ
id -Z
sudo semanage port -l
```

## Contextes de fichiers

```bash
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/www(/.*)?'
sudo restorecon -Rv /srv/www
```

`semanage fcontext` rend la règle persistante ; `chcon` est généralement temporaire et ne doit pas remplacer la définition de politique.

## Ports et booléens

```bash
sudo semanage port -a -t http_port_t -p tcp 8080
sudo getsebool -a | grep httpd
sudo setsebool -P httpd_can_network_connect_db on
```

Activer un booléen uniquement après avoir vérifié son périmètre et son impact.

## Modules

Les modules locaux doivent être petits, documentés et revus. Ne pas utiliser automatiquement `audit2allow` sans comprendre le flux attendu et la cause du refus.