# AppArmor — dépannage

## Vérifier l'activation

```bash
cat /sys/module/apparmor/parameters/enabled
sudo aa-status
```

## Un service ne démarre plus

Lire les refus, confirmer le profil associé, reproduire une seule action, puis ajouter l'autorisation minimale. Vérifier aussi les permissions DAC et les chemins de fichiers temporaires.

## Mode complain

Le mode `complain` journalise sans bloquer. Il est utile pour construire un profil, mais ne constitue pas une protection. Repasser en `enforce` après tests.

## Profil invalide

Utiliser `apparmor_parser -T -W` ou le chargement explicite pour obtenir l'erreur, corriger la syntaxe, puis recharger sans supprimer le profil validé précédent.