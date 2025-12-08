
- [[#Générer une wordlist]]
- [[#Concatener Wodlists]]
- [[#Reformatter wordlist]]
- [[#Rules]]

[[#SECLISTS CONCATENATION]]

---
# Générer une wordlist

- [[#Username Anarchy]]
- [[#CUPP]]
- [[#CRUNCH]]
- [[#PYDICTOR]]
- [[#SEQ]]
- [[#HASHCAT]]
- [[#REPOLIST]]

### CEWL

Wordlist en extrayant les données d'une page web

```bash
cewl example.com --min_word_length 5 --depth 3 --write words.txt
```

### Username Anarchy

https://github.com/urbanadventurer/username-anarchy.git

```bash
./username-anarchy  <prenom> <nom> > <file.txt>
```

A partir d'un fichier

```shell
./username-anarchy -i /home/ltnbob/names.txt > usernames
```

en precisant un format

```shell
./username-anarchy --list-formats
```

```shell
./username-anarchy -i ./names.txt  --select-format first.last > usernames
```

### CUPP

Mot de passe personnalisés avec des infos sur quelqu'un (judicieux de commencer avec peu d'informations)

```bash
cupp -i
```

### CRUNCH

8 caractères commencant par Ab suivi par 6 digits.

```shell-session
crunch 8 8 0123456789 -t Ab%%%%%% -o number_passwords.txt
```

### PYDICTOR

https://github.com/LandGrey/pydictor

Aaaa0000

```bash
python3 pydictor.py -pattern "[A-G]{1,1}<none>[a-z]{1,1}<none>[a-z]{1,1}<none>[a-z]{1,1}<none>[0-9]{1,1}<none>[0-9]{1,1}<none>[0-9]{1,1}<none>[0-9]{1,1}<none>" -o wordlistdeouf.txt

```

### SEQ

Génère une wordlist de 0000 à 9999

```bash
seq -w 0 9999 > tokens.txt
```

### HASHCAT

Mutated wordlist avec une wordlist et un fichier de règle

```bash
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```


**bons fichiers de règles:**

best64 -> /usr/share/hashcat/rules/best64.rule

**Fichiers de règles ultime :**
- https://github.com/NotSoSecure/password_cracking_rules
- https://github.com/rarecoil/pantagrule


### REPOLIST

Générer wordlist à partir d'un repo git

https://github.com/Ademking/repolist

```
repolist -u https://gihtub.com/user/repo
```

---
# Concatener Wordlists

concatener toutes les wordlists dans l'ordre indiqué et supprimer les doublons

```sh
cat wordlist1.txt wordlist2.txt wordlist3.txt | awk '!seen[$0]++' > wordlist_concat.txt
```


---
# Reformatter wordlist

### Supprimer les doublons

`seen[$0]++` garde en mémoire chaque ligne déjà vue, et n’imprime que la première occurrence.

```sh
awk '!seen[$0]++' wordlist_concat.txt > wordlist_unique.txt
```

### Supprimer les mots faisant moins de 7 caractères (inférieur ou égal à 6)

SED

```bash
cat file.txt | sed -r '/^.{,6}$/d' > seven_plus_char.txt
```

GREP

```bash
cat file.txt | grep -E '.{8}' > seven_plus_char.txt
```

### Supprimer les mots plus de 7 caractères (supérieur ou égal à 8)

```bash
cat file.txt | sed -r '/^.{8,}$/d' > seven_minus_char.txt
```
### Supprimer les mots n'ayant pas de caractères spéciaux

SED

```bash
cat file.txt | sed -r '/[!-/:-@\[-`\{-~]+/!d' > special_char.txt
```

### Supprimer les mots n'ayant pas de chiffres

SED

```bash
cat file.txt | sed -r '/[0-9]+/!d' > Number.txt
```

GREP

```BASH
grep '[[:digit:]]'
```
### Supprimer les mots n'ayant pas de minuscules

SED

```bash
cat file.txt | sed -r '/[a-z]+/!d' > Maj.txt
```

GREP

```BASH
grep '[[:lower:]]'
```

### Supprimer les mots n'ayant pas de majuscules

SED

```bash
cat file.txt | sed -r '/[A-Z]+/!d' > Min.txt
```

GREP

```bash
grep '[[:upper:]]'
```

### Récupérer les mots commençants par A

```bash
grep -oE '\bA\S*' password.list > passwordA.list 
```

---
# Rules

default rules

```shell
ls -l /usr/share/hashcat/rules/
```

Fichiers de règles ultime :
- https://github.com/NotSoSecure/password_cracking_rules
- https://github.com/stealthsploit/OneRuleToRuleThemStill
- https://github.com/rarecoil/pantagrule

| **Rule**                    | **Description**                                | **Link**                                                                                              |
| --------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `one-rule-to-rule-them-all` | One super rule that tops all the tests         | [Link](https://github.com/stealthsploit/OneRuleToRuleThemStill/blob/main/OneRuleToRuleThemStill.rule) |
| `rules-collection`          | Largest collection of hashcat rule-files       | [Link](https://github.com/n0kovo/hashcat-rules-collection)                                            |
| `nsa-rules`                 | Rules generated from cracked passwords         | [Link](https://github.com/NSAKEY/nsa-rules)                                                           |
| `Hob0Rules`                 | Rule based on statistics and industry patterns | [Link](https://github.com/praetorian-inc/Hob0Rules)                                                   |


## Générer un fichier de règles

```shell
echo 'c so0 si1 se3 ss5 sa@ $2 $0 $1 $9' > rule.txt
```

| **Function**    | **Description**                                                    | **Input**                             | **Output**                                                                                                        |
| --------------- | ------------------------------------------------------------------ | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| l               | Convert all letters to lowercase                                   | InlaneFreight2020                     | inlanefreight2020                                                                                                 |
| u               | Convert all letters to uppercase                                   | InlaneFreight2020                     | INLANEFREIGHT2020                                                                                                 |
| c / C           | capitalize / lowercase first letter and invert the rest            | inlaneFreight2020 / Inlanefreight2020 | Inlanefreight2020 / iNLANEFREIGHT2020                                                                             |
| t / TN          | Toggle case : whole word / at position N                           | InlaneFreight2020                     | iNLANEfREIGHT2020                                                                                                 |
| d / q / zN / ZN | Duplicate word / all characters / first character / last character | InlaneFreight2020                     | InlaneFreight2020InlaneFreight2020 / IInnllaanneeFFrreeiigghhtt22002200 / IInlaneFreight2020 / InlaneFreight20200 |
| { / }           | Rotate word left / right                                           | InlaneFreight2020                     | nlaneFreight2020I / 0InlaneFreight202                                                                             |
| ^X / $X         | Prepend / Append character X                                       | InlaneFreight2020 (^! / $! )          | !InlaneFreight2020 / InlaneFreight2020!                                                                           |
| r               | Reverse                                                            | InlaneFreight2020                     | 0202thgierFenalnI                                                                                                 |
| s               | Substitute                                                         | InlaneFreight2020 (se3)               | Inlan3Fr3ight2020                                                                                                 |
|                 |                                                                    |                                       |                                                                                                                   |


---
# SECLISTS CONCATENATION


## DNS / Subdomains

```sh
cat /usr/share/seclists/Discovery/DNS/namelist.txt /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt /usr/share/seclists/Discovery/DNS/dns-Jhaddix.txt /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt /usr/share/seclists/Discovery/DNS/bug-bounty-program-subdomains-trickest-inventory.txt /usr/share/seclists/Discovery/DNS/services-names.txt /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt /usr/share/seclists/Discovery/DNS/tlds.txt /usr/share/seclists/Discovery/DNS/deepmagic.com-prefixes-top50000.txt /usr/share/seclists/Discovery/DNS/FUZZSUBS_CYFARE_1.txt /usr/share/seclists/Discovery/DNS/FUZZSUBS_CYFARE_2.txt /usr/share/seclists/Discovery/DNS/shubs-stackoverflow.txt /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt /usr/share/seclists/Discovery/DNS/subdomains-spanish.txt /usr/share/seclists/Discovery/DNS/italian-subdomains.txt /usr/share/seclists/Discovery/DNS/sortedcombined-knock-dnsrecon-fierce-reconng.txt /usr/share/seclists/Discovery/DNS/combined_subdomains.txt | awk '!seen[$0]++' > DNS_concat.txt
```

## DIRECTORIES & FILES

```sh
cat /usr/share/seclists/Discovery/Web-Content/quickhits.txt /usr/share/seclists/Discovery/Web-Content/common.txt /usr/share/wordlists/dirb/big.txt /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt /usr/share/seclists/Discovery/Web-Content/raft-large-extensions-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-large-extensions.txt /usr/share/seclists/Discovery/Web-Content/raft-large-files-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt /usr/share/seclists/Discovery/Web-Content/raft-large-words-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-extensions-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-extensions.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt /usr/share/seclists/Discovery/Web-Content/raft-small-extensions-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-small-extensions.txt /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt | awk '!seen[$0]++' > 'DIR&FILES_concat.txt'
```

## PARAMETERS

```sh
wget https://wordlists-cdn.assetnote.io/data/automated/httparchive_parameters_top_1m_2025_08_27.txt
wget https://raw.githubusercontent.com/s0md3v/Arjun/refs/heads/master/arjun/db/small.txt
wget https://raw.githubusercontent.com/s0md3v/Arjun/refs/heads/master/arjun/db/medium.txt
wget https://raw.githubusercontent.com/s0md3v/Arjun/refs/heads/master/arjun/db/large.txt
wget https://gist.githubusercontent.com/nullenc0de/9cb36260207924f8e1787279a05eb773/raw/0197d33c073a04933c5c1e2c41f447d74d2e435b/params.txt
cat /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt httparchive_parameters_top_1m_2025_08_27.txt small.txt medium.txt large.txt params.txt | awk '!seen[$0]++' > PARAM_concat.txt
```

