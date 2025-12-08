

- [[#Modules]]
- [[#Scripts classiques]]

#### PowerShell Extensions

| **Extension** | **Description**                |
| ------------- | ------------------------------ |
| ps1           | Script.                        |
| psm1          | Fichier de module.             |
| psd1          | Manifest detaillant le script. |
# Modules

> [!TIP]
> Ensemble de scripts

4 composants essentiels:

1. Répertoire contenant les fichiers et le contenu dans $env:PSModulePath
2. Un fichier Manifest qui liste tous les fichiers et informations pertinentes à propos du module et de ses fonctions (scripts, dépendances etc.).
3. Des scripts powershell ou fichieres de modulee contenant les fonctions.
4. Ressources en support aux modules et scripts.
-> Peut également être un simple .psm1 sans manifest ou fichiers supplémentaires.

## Manifest

- Décrire le contenu et les attributs du module.
- Définir les conditions préalables. ( modules spécifiques extérieurs au module lui-même, variables, fonctions, etc.)
- Déterminer comment les composants sont traités.

Contient:

- Métadonnées sur le module, telles que le numéro de version du module, l'auteur et la description.
- les conditions préalables nécessaires à l'importation du module, telles que la version de Windows PowerShell, la version du Common Language Runtime (CLR) et les modules requis
- Les directives de traitement, telles que les scripts, les formats et les types à traiter.
- Restrictions sur les membres du module à exporter, telles que les alias, les fonctions, les variables et les cmdlets à exporter.

#### Générer un Manifest

```powershell
New-ModuleManifest -Path path/to/module.psd1 -PassThru
```

## Créer un fichier de module

```powershell
ni module.psm1 -ItemType File
```

#### Importer un module externe

```powershell
Import-Module ActiveDirectory 
```

## Variables

Variables déclarées commencent par $

```powershell
$Hostname = $env:ComputerName
$IP = ipconfig 
$Domain = Get-ADDomain  
$Users = Get-ChildItem C:\Users\ 
```

| PORTÉE DE VARIABLE | DESCRIPTION                                                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Global**         | La variable est accessible partout dans la session PowerShell, y compris en dehors du script actuel. C'est la portée par défaut pour les variables créées dans une console PowerShell. Elle est également associée aux variables automatiques.                                                 |
| **Local**          | La variable est accessible uniquement à l'intérieur de la fonction ou du script où elle a été définie. C'est la portée attribuée par défaut à une variable dans un script (à moins qu'un nom de portée soit indiqué). Lorsque le script a terminé son exécution, la variable est inaccessible. |
| **Script**         | La variable est accessible partout dans le script actuel, mais pas en dehors de celui-ci.                                                                                                                                                                                                      |
```
$scope:variable
```

#### Constantes

```powershell
Set-Variable variable -Option Constant -Value "valeur"
```

## Fonctions

Afficher l'output des variables dans un autre fichier généré.

```powershell
function Get-Recon {  

    $Hostname = $env:ComputerName  

    $IP = ipconfig

    $Domain = Get-ADDomain 

    $Users = Get-ChildItem C:\Users\

    new-Item ~\Desktop\recon.txt -ItemType File 

    $Vars = "***---Hostname info---***", $Hostname, "***---Domain Info---***", $Domain, "***---IP INFO---***",  $IP, "***---USERS---***", $Users

    Add-Content ~\Desktop\recon.txt $Vars
  } 
```

## Commentaires

```powershell
# Commenter une ligne

<# Commenter
un bloc de ligne #>
```

## Inclure de l'aide

Commenter la description et inclure des mots clés (.keyword) -> permet d'utiliser Get-Help sur le module

```powershell
<# 
.Description  
This function performs some simple recon tasks for the user. We import the module and issue the 'Get-Recon' command to retrieve our output. Each variable and line within the function and script are commented for our understanding. Right now, this module will only work on the local host from which you run it, and the output will be sent to a file named 'recon.txt' on the Desktop of the user who opened the shell. Remote Recon functions are coming soon!  

.Example  
After importing the module run "Get-Recon"
'Get-Recon


    Directory: C:\Users\MTanaka\Desktop


Mode                 LastWriteTime         Length Name                                                                                                                                        
----                 -------------         ------ ----                                                                                                                                        
-a----         11/3/2022  12:46 PM              0 recon.txt '

.Notes  
Remote Recon functions coming soon! This script serves as our initial introduction to writing functions and scripts and making PowerShell modules.  

#>
```

## Protéger les fonctions
#### Empêcher toutes les fonctions du module d'être exportées

```powershell
Export-ModuleMember  
```

#### Autoriser l'export d'une fonction et d'une variable

```powershell
Export-ModuleMember -Function Get-Recon -Variable Hostname 
```

#### Autoriser l'export de toutes les fonctions

```powershell
Export-ModuleMember -Function * -Variable Hostname 
```


## Scope

#### Scope Levels

| **Scope** | **Description**                                                                                                                                                                                                                                                                                |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Global    | Il s'agit du scope par défaut de PowerShell. Il affecte tous les objets qui existent lorsque PowerShell démarre ou qu'une nouvelle session est ouverte. Les variables, les alias, les fonctions et tout ce que vous spécifiez dans votre profil PowerShell seront créés dans le scope globale. |
| Local     | Il s'agit du scope actuel dans laquelle vous travaillez. Il peut s'agir de n'importe quel scope par défaut ou d'un scope enfant créé.                                                                                                                                                          |
| Script    | Il s'agit d'un scope temporaire qui s'applique à tous les scripts en cours d'exécution. Il ne s'applique qu'au script et à son contenu. Les autres scripts et tout ce qui se trouve à l'extérieur ne sauront pas qu'il existe. Pour le script, sa portée est la portée locale.                 |

Possible de créer des scopes enfants -> https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scopes?view=powershell-7.5&viewFallbackFrom=powershell-7.2

---
# Scripts classiques


- [[#Arguments ($Args[0] - $Args[31])]]
- [[#Paramètres]]
- [[#Arrays]]
- [[#Entrée / Sortie]]
- [[#Boucles]]
- [[#Structures conditionnelles]]


https://ss64.com/ps/syntax-args.html

Crescendo permet de générer des cmdlets
https://github.com/PowerShell/Crescendo


## Arguments ($Args[0] - $Args[31])

```powershell
$Args[0] -> argument 1
$Args[1] -> argument 2
...
$Args[31] -> argument 32
```

## Paramètres

doit apparaitre en premier dans le script ou la fonction si utilisé

```powershell
param (
   [string]$welcome, 
   [string]$ComputerName = $env:computername,    
   [string]$username = $(throw "-username is required."),
   [string]$password = $( [Read-Host](https://ss64.com/ps/read-host.html) -asSecureString "Input password" ),
   [switch]$SaveData = $false
)
```

## Arrays

```powershell
$fruits = @(“pomme”, “banane”, “orange”)
```

## Entrée / Sortie


## Boucles

### Boucle FOR

```POWERSHELL
for ($i = 0 ; $i -lt 10 ; $i++)
{
	echo $i
}
```

### Boucle FOREACH

```powershell
foreach ($item in $collection)
{
	Write-Output $item
}
```

### Boucle WHILE

```powershell
while ($condition)
{
	# Votre code ici
}
```

### Boucle DO WHILE (s'exécute au moins une fois)

```powershell
do 
{ 
	# Votre code ici 
}
while ($condition)
```

### Boucle DO UNTIL (s'exécute au moins une fois)

```powershell
do 
{ 
	# Votre code ici 
}
until ($condition)
```


## Structures conditionnelles

https://www.it-connect.fr/chapitres/powershell-switch-les-structures-conditionnelles/
### If, ElseIf, Else

```powershell
If("<condition 1>")
{ 
	# bloc de code (instructions)
}
ElseIf("<condition 2>")
{
	# bloc de code (instructions)
}
Else
{
	# bloc de code (instructions)
}
```

### Switch

```powershell
$NomLogiciel = Read-Host -Prompt "Quel logiciel souhaitez-vous lancer ?"

Switch ($NomLogiciel)
{ 
	"logiciel 1" { Start-Process logiciel 1 }
	"logiciel 2" { Start-Process logiciel 2 }
	"logiciel 3" { Start-Process logiciel 3 }
	Default { "Désolé, je n'ai pas trouvé ce logiciel" }
}
```

