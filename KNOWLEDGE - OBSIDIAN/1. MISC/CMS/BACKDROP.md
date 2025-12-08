


>[!SUMMARY] Table of Contents
>- [[BACKDROP#ENUMERATION MANUELLE|ENUMERATION MANUELLE]]
>    - [[BACKDROP#Identifier Drupal|Identifier Drupal]]
>- [[BACKDROP#EXPLOITATION|EXPLOITATION]]
>    - [[BACKDROP#Tester Weak credentials / Credential stuffing|Tester Weak credentials / Credential stuffing]]
>        - [[BACKDROP#Credential Stuffing (Liste de default creds)|Credential Stuffing (Liste de default creds)]]
>    - [[BACKDROP#Uploading a Backdoored Module|Uploading a Backdoored Module]]



# ENUMERATION MANUELLE

- [[#Identifier Drupal]]
- [[#Version Drupal]]


/robots.txt
/sitemap.xml

## Identifier BackDrop

```shell
curl -s http://inlanefreight.local | grep BackDrop
```

contenu indexé au travers de nodes.

```
/node/<nodeid>
```

---

# EXPLOITATION

- [[#Tester Weak credentials / Credential stuffing]]
- [[#Uploading a Backdoored Module]]


## Tester Weak credentials / Credential stuffing

admin:admin, admin:password....
### Credential Stuffing (Liste de default creds)

Resources:

https://www.cirt.net/passwords
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv
https://github.com/ihebski/DefaultCreds-cheat-sheet/tree/main


## Uploading a Backdoored Module

Authenticated RCE -> https://www.exploit-db.com/exploits/52021

