
> [!TIP]
> - Copie de fichiers volumineux
> - Créer une copie de sauvegarde de données
> - Synchroniser des données entre deux machines
> - Copie de fichiers en miroir

https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/robocopy?WT.mc_id=AZ-MVP-5004580

OPTIONS

- **`/S`** – Copier les sous-répertoires de l'arborescence, à l'exception des répertoires vides.
- **`/E`** – Copier les sous-répertoires de l'arborescence, y compris les vides.
- **`/MIR`** - Copier avec le mode miroir.
- **`/SEC`** - Copier les permissions de la source vers la destination.
- **`/COPYALL`** - Copier tous les attributs d'un fichier (date de création, date de modification, etc.).
- **`/Z`** – Copier les fichiers en mode redémarrable, c'est-à-dire qu'en cas d'interruption, Robocopy peut reprendre là où il s’est arrêté au lieu de recopier l’intégralité du fichier.
- **`/ZB`** – Utiliser le mode redémarrable. Si l’accès est refusé, utilise le mode de sauvegarde.
- **`/R:5`** – Réessayer 5 fois en cas d’échec (la valeur par défaut est 1 million, donc c'est important de personnaliser cette option).
- **`/W:5`** – Attendre 5 secondes entre chaque essai (par défaut : 30 secondes).
- **`/NP`** – N’affiche pas le pourcentage copié.
- **`/V`** – Mode verbeux pour afficher une sortie plus détaillée, notamment avec les fichiers ignorés.
- **`/MT:16`** – Effectuer des copies multithreadées, avec 16 threads dans cet exemple (par défaut 8 – maximum : 128).
- **`/LOG`** - Préciser le chemin vers un fichier de log


```POWERSHELL
robocopy /?
```

### Copie de tous les dossiers et sous dossiers y compris vide en mode mirroir

```powershell
robocopy <Source> <Destination> <Fichiers> /E /MIR /SEC /R:5 /W:5 /DCOPY:DAT
```

### Vérifier l'intégrité du dossier copié

Calcul du hash

```powershell
$Source = (Get-ChildItem <path/to/dossier_source> -Recurse | Get-FileHash -Algorithm MD5).Hash | Out-String
$Destination = (Get-ChildItem <path/to/dossier_destination> -Recurse | Get-FileHash -Algorithm MD5).Hash | Out-String
Get-FileHash -InputStream ([IO.MemoryStream]::new([char[]]$Source))
Get-FileHash -InputStream ([IO.MemoryStream]::new([char[]]$Destination))
```

