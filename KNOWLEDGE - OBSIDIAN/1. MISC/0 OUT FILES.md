
## LINUX

Réécris 5x le contenu du fichier avec de la data random avant de tout mettre à 0 et supprimer le fichier.

```bash
shred -n 5 -z -u <filename>
```


## WINDOWS

1. Supprimer définitivement le fichier dans un premier temps
2. WipeOut l'espace non utilisé avec cipher

```Powershell
cipher /w:PathName
```
