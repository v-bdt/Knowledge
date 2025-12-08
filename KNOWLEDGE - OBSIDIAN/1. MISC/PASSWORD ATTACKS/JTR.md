
> [!TIP]
> Exploite le CPU

- [[#Attaque Single mode]]
- [[#Attaque par dictionnaire]]
- [[#Attaque Incrémentale]]
- [[#Crack de fichiers]]



Identifier format de hash correspondant a john

```shell
hashid -j 193069ceb0461e1d40d216e32c79c704 
```


---
# Attaque Single mode

Utile pour les mots de passe linux. Génère une wordliste à la volée en se basant sur le username, le nom du répertoire home et les valeurs GECOS (full name, room number, phone number, etc.). -> les mots générés passent par un fichier de règle.

```shell
john --single unshadowed
```



---
# Attaque par dictionnaire

Wordlist + rules

```bash
john --format=<hash_type> <hash or hash_file> --wordlist=/usr/share/wordlists/rockyou.txt --rules=~/Download/OneRuleToRuleThemAll.txt
```

---

# Attaque Incrémentale

Génère les caractères à la volée et teste toute les combinaisons possibles (consomme énormément de ressources et de temps)

```bash
john --incremental <hash_file>
```

---
# Crack de fichiers


```shell
locate *2john*
```

```shell
<tool> <file_to_crack> > file.hash
```

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |

- [[#Clé SSH chiffrée]]
- [[#Document Office protégé]]
- [[#Archive protégée]]
- [[#Archive chiffrée avec OpenSSL]]
- [[#BitLocker]]

## Clé SSH chiffrée
#### Mettre clé SSH au format john

```shell
ssh2john SSH.private > ssh.hash
```

#### Crack de la clé

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash
```

```shell
john ssh.hash --show
```

## Document Office protégé

#### Mettre document au format john

```shell
office2john.py Protected.docx > protected-docx.hash
```

#### Crack du mot de passe

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt protected-docx.hash
```

```shell-session
john protected-docx.hash --show
```

## Archive protégée

```shell
zip2john ZIP.zip > zip.hash
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash
```

## Archive chiffrée avec OpenSSL

for loop avec la wordlist

```shell
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

```shell
ls
```

## BitLocker

```shell
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt backup.hash
```




