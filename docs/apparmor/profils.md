# AppArmor — profils et règles

## Structure minimale

```text
#include <tunables/global>
/usr/local/bin/mon-service {
  #include <abstractions/base>
  /etc/mon-service/** r,
  /var/lib/mon-service/** rw,
  /run/mon-service.sock rw,
  network inet stream,
  capability setgid,
}
```

Les permissions de fichier courantes sont `r`, `w`, `a`, `k`, `m`, `l` et `ix`/`px`/`cx` pour les transitions d'exécution. Déclarer seulement les accès nécessaires.

## Héritage et transitions

- `ix` : exécuter avec le profil courant ;
- `px` : transition vers un profil nommé ;
- `cx` : transition vers un sous-profil.

Les abstractions facilitent la maintenance mais doivent être relues : elles peuvent autoriser davantage que prévu. Les variables de `tunables` évitent les chemins codés en dur.