

Afficher les colonnes 6 et 5 (s'affiche dans l'ordre indiqué)

```sh
awk '{print $6, $5}'
```

Supprimer les doublons

```sh
awk '!seen[$0]++'
```