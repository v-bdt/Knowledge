
[[#ENUMERATION MANUELLE]]
[[#EXPLOITATION]]


1. Créer un compte si possible

# ENUMERATION MANUELLE

## Identifier version

```sh
curl -s http://code.fries.htb/ | grep "Powered by Gitea" -A 10
```

## Creds Hunting

/explore -> voir les projets publiques (si non connecté) ou les projets privés (si connecté)

Faire un git clone des projets et chercher des identifiants avec TruffleHog [[9. GIT]]


# EXPLOITATION


# POST-EXPLOITATION

- [[#Identifiants en base de données]]
- [[#Hash crack]]



### Identifiants en base de données

Récupérer les identifiants en base de données

```
SELECT email,salt,passwd,passwd_hash_algo FROM "user";
```

### Hash crack

Crack des identifiants git récupérés en base de données
https://www.unix-ninja.com/p/cracking_giteas_pbkdf2_password_hashes


1. Mise au format hashcat
https://raw.githubusercontent.com/hashcat/hashcat/refs/heads/master/tools/gitea2hashcat.py

```sh
python3 gitea2hashcat.py "<salt>|<passwd>" > git.hashes
```

2. Crack avec hashcat (mode 10900)

```sh
hashcat -m 10900 git.hashes /usr/share/wordlists/rockyou.txt
```