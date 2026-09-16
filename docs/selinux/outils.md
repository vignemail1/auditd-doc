# Outils CLI SELinux

Les commandes proviennent principalement de `policycoreutils`, `selinux-utils`, `selinux-policy`, `setroubleshoot` et `policycoreutils-python-utils`.

## `getenforce`, `setenforce`, `sestatus`

```bash
getenforce
sudo setenforce 0       # permissive temporaire
sudo setenforce 1       # enforcing temporaire
sestatus
sestatus -b             # booléens
sestatus -v             # contextes des fichiers/processus
```

`setenforce` ne persiste pas après redémarrage ; modifier `/etc/selinux/config` pour le mode par défaut. Ne pas désactiver SELinux pour masquer un problème sans analyse préalable.

## `semanage`

```bash
sudo semanage fcontext -l
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/www(/.*)?'
sudo semanage fcontext -m -t httpd_sys_rw_content_t '/srv/www/data(/.*)?'
sudo semanage fcontext -d '/srv/www(/.*)?'
sudo restorecon -Rv /srv/www
sudo semanage port -l | grep http
sudo semanage port -a -t http_port_t -p tcp 8080
sudo semanage port -d -t http_port_t -p tcp 8080
sudo semanage boolean -l
sudo semanage boolean -m --on httpd_can_network_connect
```

`-a`, `-m`, `-d` ajoutent, modifient ou suppriment une entrée. Les expressions de chemins doivent être citées. `semanage` modifie la politique locale persistante ; `restorecon` applique ensuite les contextes aux objets existants.

## `chcon` et `restorecon`

```bash
sudo chcon -t httpd_sys_content_t /srv/www/index.html
sudo chcon -R -t httpd_sys_content_t /srv/www
sudo chcon --reference=/var/www/html/index.html /srv/www/index.html
sudo restorecon -v /srv/www/index.html
sudo restorecon -RFv /srv/www
sudo restorecon -nRv /srv/www       # simulation selon version
```

`chcon` modifie directement le contexte et peut être écrasé. Préférer `semanage fcontext` puis `restorecon`. `-R` est récursif, `-v` verbeux, `-F` force la réapplication et `-n` évite la modification lorsqu’il est disponible.

## `ausearch`, `sealert`, `audit2why`, `audit2allow`

```bash
sudo ausearch -m AVC -ts recent -i
sudo ausearch -m AVC -c nginx --raw | audit2why
sudo sealert -a /var/log/audit/audit.log
sudo ausearch -m AVC -ts today --raw | audit2allow -w
sudo ausearch -m AVC -ts today --raw | audit2allow -M local_http
sudo semodule -i local_http.pp
```

`audit2why` explique un refus. `audit2allow` génère une proposition de règle ; ne jamais l’installer sans comprendre le besoin, vérifier le type de domaine, les chemins, les booléens et le principe du moindre privilège. Le fichier `.te` généré peut être relu avant compilation.

## `sesearch`, `seinfo`, `sesearch`

```bash
seinfo -t | grep httpd
seinfo -x -a
sesearch -A -s httpd_t -t httpd_sys_content_t -c file -p read
sesearch -A -s httpd_t -c tcp_socket -p name_connect
```

`seinfo` interroge les composants de la politique ; `sesearch` recherche les règles autorisant ou refusant une interaction. Ajouter `-C` pour tenir compte des contraintes selon la version.

## `semanage permissive` et modules

```bash
sudo semanage permissive -l
sudo semanage permissive -a myapp_t
sudo semanage permissive -d myapp_t
sudo semodule -l
sudo semodule -i local_http.pp
sudo semodule -r local_http
sudo semodule -B
```

Rendre un domaine permissif est une mesure de diagnostic ciblée, différente du mode global permissive. `semodule -B` reconstruit la politique ; planifier cette opération sur les systèmes sensibles.

## Manpages et références

```bash
man getenforce
man setenforce
man semanage
man restorecon
man chcon
man ausearch
man audit2why
man audit2allow
man sesearch
man seinfo
man semodule
man sealert
```

- [SELinux Project](https://selinuxproject.org/)
- [Red Hat — Using SELinux](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/)
- [Fedora SELinux User Guide](https://docs.fedoraproject.org/en-US/quick-docs/selinux-changing-states-and-modes/)
- [SELinux man pages](https://man7.org/linux/man-pages/dir_section_8.html)
- [SELinux userspace](https://github.com/SELinuxProject/selinux)
