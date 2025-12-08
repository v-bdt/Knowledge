
https://github.com/rasta-mouse/Sherlock


> [!TIP]
> Identifier les CVEs possibles en fonctions des patchs manquants


1. Changer la politique d'execution des scripts

```powershell
Set-ExecutionPolicy Bypass -Scope Process
```

2. Importer module

```powershell-session
Import-Module .\Sherlock.ps1
```

3. Run tous les checks

```powershell-session
Find-AllVulns
```