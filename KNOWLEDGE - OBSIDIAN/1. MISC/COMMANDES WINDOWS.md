
# POWERSHELL

#### S'authentifier avec un compte

```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)
```

#### PsRemoting

```powershell
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $Cred
```

installation du module sous linux

```bash
sudo pwsh -Command 'Install-Module -Name PSWSMan'
sudo pwsh -Command 'Install-WSMan'
```

#### Afficher le current path

```powershell
pwd
```

```powershell
Get-Location
```

#### Lister les fichiers

```powershell
ls <path>
```

```powershell
Get-ChildItem <path>
```

#### Lire un fichier

```powershell
cat <fichier>
```

```powershell
Get-Content <fichier>
```

#### Lancer un processus en tant qu'autre user pour accès à distance

```powershell
runas /netonly /user:david "cmd.exe"
```


### PARTAGES #POWERSHELL

#### Monter un partage

sans creds

```powershell
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

avec creds

```powershell
$username = 'plaintext'
$password = 'Password123'
$secpassword = ConvertTo-SecureString $password -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
```

#### Identifier le nombre de fichiers dans le partage

```powershell
(Get-ChildItem -File -Recurse | Measure-Object).Count
```

#### Rechercher les fichiers contenant un string dans le nom

```powershell
Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
```

#### Rechercher un string dans les fichiers

```powershell
Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```




---

# CMD

#### Afficher le current path

```cmd
cd
```

#### Lister les fichiers

```cmd
dir <path>
```

#### Lire un fichier

```cmd
MORE <fichier>
```

#### Activer compte utilisateur

```
net user <user> /active:yes
```


### PARTAGES #CMD

#### Monter un partage

```cmd
net use n: \\192.168.220.129\Finance
```

```cmd-session
net use n: \\192.168.220.129\Finance /user:plaintext Password123
```

#### Identifier le nombre de fichiers dans le partage

```cmd
 dir n: /a-d /s /b | find /c ":\"
```

#### Rechercher un fichier contenant un string dans le nom

```cmd
dir n:\*<string>* /s /b
```
#### Rechercher un string dans les fichiers

```cmd
findstr /s /i <string> n:\*.*
```


