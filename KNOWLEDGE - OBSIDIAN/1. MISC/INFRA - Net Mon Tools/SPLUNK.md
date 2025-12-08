
> [!TIP]
> Fonctionne avec API REST
> Est installé avec Python par défaut et donc permet d'éxecuter des scripts python.

- [ ] [[#ENUMERATION MANUELLE]]
- [ ] [[#EXPLOITATION]]
- [ ] [[#HARDENING]]



port 8000 et 8089 par défaut


# ENUMERATION MANUELLE

Page de login

https://10.129.201.50:8000/en-US/account/login?return_to=%2Fen-US%2F

## Tester default creds

 `admin:changeme`
 `admin`, `Welcome`, `Welcome1`, `Password123`...


---

# EXPLOITATION

- [[#RCE]]
- [[#CVEs]]

## RCE

https://github.com/0xjpuff/reverse_shell_splunk

1. Modifier run.ps1 ou rev.py et y mettre ip / port attacker
2. Modifier input.conf et garder uniquement le run.bat ou rev.py
3. Créer un tarball des fichiers

```shell-session
tar -cvzf updater.tar.gz splunk_shell/
```

3. Lancer listener
4. Dans Splunk -> Apps -> Manage Apps -> Install app from file -> Browse et choisir le tar
5. Upload



## CVEs

Chercher CVEs


---

# HARDENING

https://docs.splunk.com/Documentation/Splunk/8.2.2/Security/Hardeningstandards