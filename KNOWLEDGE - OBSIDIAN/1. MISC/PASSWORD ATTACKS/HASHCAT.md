

> [!Warning]
> Dans le cas d'un hash récupéré dans une base de donnée Web, tester de cracker le hash d'un compte qu'on aurait crée afin de vérifier qu'on a choisi le bon mode.


- [[#Attaque par dictionnaire]]
- [[#Attaque par combinaison de dictionnaires]]
- [[#Attaque par mask]]
- [[#Attaque Hybride]]
- [[#Optimiser les performances]]
- [[#Rules]]
- [[#Formats]]

- [[#Crack de fichiers]]

---
# Attaque par dictionnaire

> [!TIP]
> Tester avec et sans flag -S (peut être beaucoup plus rapide ou plus lent)

Wordlist + rules

```bash
hashcat -a 0 -m <hash type> <hash file> <wordlist> -r <rulelist>
```

---
# Attaque par combinaison de dictionnaires

Combiner des wordlists

```bash
hashcat -a 1 -m <hash type> <hash file> <wordlist1> <wordlist2> ...
```

---
# Attaque par mask

> [!TIP]
> Il n'est pas rare de retrouver des mots de passe tel que Entreprise2022! par exemple

Permet de spécifier un pattern.

| **Placeholder** | **Meaning**                                             |
| --------------- | ------------------------------------------------------- |
| ?l              | lower-case ASCII letters (a-z)                          |
| ?u              | upper-case ASCII letters (A-Z)                          |
| ?d              | digits (0-9)                                            |
| ?h              | 0123456789abcdef                                        |
| ?H              | 0123456789ABCDEF                                        |
| ?s              | special characters («space»!"#$%&'()*+,-./:;<=>?@[]^_`{ |
| ?a              | ?l?u?d?s                                                |
| ?b              | 0x00 - 0xff                                             |

```shell
hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

Les options -1 à -4 permettent de customiser un placeholder (par ex -1 012 testera 0 1 et 2 à l'emplacement ou ?1 sera situé).

```shell
hashcat -a 3 -m 0 md5_mask_example_hash -1 012 'ILFREIGHT?l?l?l?l?l20?1?d'
```

Possible de mettre les custom placeholder dans un fichier .hcchr

```shell
echo 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789' > starterchars.hcchr
echo 'abcdefghijklmnopqrstuvwxyz!@#$%&*-+0123456789' > endchars.hcchr
```

```shell
hashcat -a 3 -m 22000 wpa_hash --increment --increment-min=8 --increment-max=12 -1 starterchars.hcchr -2 endchars.hcchr ?1?1?1?1?1?1?2?2?2?2?2?2
```

 Possible d'incrémenter avec --increment

```shell
hashcat -a 3 -m 22000 wpa_hash --increment --increment-min 8 --increment-max=14 '?u?l?l?l?l?l?l?l?d?d?d?d?d?d'
```




---
# Attaque Hybride


Mélange d'attaque par wordlist en premier + mask -> wordmask

```shell
hashcat -a 6 -m 0 hybrid_hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt '?d?s'
```

Mélange d'attaque par mask en premier + wordlist -> maskword

```shell
hashcat -a 7 -m 0 hybrid_hash '?d?s' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt
```


---

# Optimiser les performances

> [!TIP]
> Exploite le GPU par défaut -> argument '-D 1' pour utiliser le CPU (+ lent).
> Argument -d pour spécifier le device id à utiliser
> Possible d'ajuster le workload avec -w de 1 à 4.
> Possible d'optimer le kernel avec -O ou --optimized-kernel-enable.
> Combiner --optimized-kernel-enable -w 4 sur le GPU pour cracker le plus vite possible.

[[#GPU Based]]
[[#CPU Based]]
[[#GPU + CPU]]


Identifier les CPU et GPU disponibles

```shell-session
 hashcat -I
```

## GPU Based

Vitesse maximum possible avec l'optimisation du kernel et le workload à 4 sur le GPU.

```shell-session
hashcat -m 22000 hash /opt/wordlist.txt -D 2 --optimized-kernel-enable -w 4
```

## CPU Based


Forcer sur le CPU

```shell-session
hashcat -m 22000 hash /opt/wordlist.txt -D 1
```

Specifier les coeurs utilisés

```shell-session
hashcat -m 22000 hash /opt/wordlist.txt -D 1 --cpu-affinity=1,2,3,4
```

Specifier le nb de threads

```shell-session
hashcat -m 22000 hash /opt/wordlist.txt -D 1 --cpu-affinity=1,2,3,4 --hook-threads=8
```


## GPU + CPU

Possible de combiner CPU et GPU (-D 1,2) avec toutes les options pour optimiser

```shell-session
hashcat -m 22000 hash /opt/wordlist.txt -D 1,2 -d 1,2 --cpu-affinity=1,2,3,4 --hook-threads=8 --optimized-kernel-enable -w 4
```

---
# Rules

default rules

```shell
ls -l /usr/share/hashcat/rules/
```

Fichiers de règles ultime :
- https://github.com/NotSoSecure/password_cracking_rules
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

|**Function**|**Description**|**Input**|**Output**|
|---|---|---|---|
|l|Convert all letters to lowercase|InlaneFreight2020|inlanefreight2020|
|u|Convert all letters to uppercase|InlaneFreight2020|INLANEFREIGHT2020|
|c / C|capitalize / lowercase first letter and invert the rest|inlaneFreight2020 / Inlanefreight2020|Inlanefreight2020 / iNLANEFREIGHT2020|
|t / TN|Toggle case : whole word / at position N|InlaneFreight2020|iNLANEfREIGHT2020|
|d / q / zN / ZN|Duplicate word / all characters / first character / last character|InlaneFreight2020|InlaneFreight2020InlaneFreight2020 / IInnllaanneeFFrreeiigghhtt22002200 / IInlaneFreight2020 / InlaneFreight20200|
|{ / }|Rotate word left / right|InlaneFreight2020|nlaneFreight2020I / 0InlaneFreight202|
|^X / $X|Prepend / Append character X|InlaneFreight2020 (^! / $! )|!InlaneFreight2020 / InlaneFreight2020!|
|r|Reverse|InlaneFreight2020|0202thgierFenalnI|


## Rejection Rules

https://hashcat.net/wiki/doku.php?id=rule_based_attack#rules_used_to_reject_plains


NOTE: Reject rules only work either with hashcat-legacy, or when using “-j” or “-k” with hashcat. They will not work as regular rules (in a rule file) with hashcat.

|Name|Function|Description|Example Rule|Note|
|---|---|---|---|---|
|Reject less|<N|Reject plains if their length is greater than N|<G|*|
|Reject greater|>N|Reject plains if their length is less than N|>8|*|
|Reject equal|_N|Reject plains of length not equal to N|_7|*|
|Reject contain|!X|Reject plains which contain char X|!z||
|Reject not contain|/X|Reject plains which do not contain char X|/e||
|Reject equal first|(X|Reject plains which do not start with X|(h||
|Reject equal last|)X|Reject plains which do not end with X|)t||
|Reject equal at|=NX|Reject plains which do not have char X at position N|=1a|*|
|Reject contains|%NX|Reject plains which contain char X less than N times|%2a|*|
|Reject contains|Q|Reject plains where the memory saved matches current word|rMrQ|e.g. for palindrome|

Rejeter les candidats inferieurs à 8 caractères et supérieurs a 63 caractères.

```sh
hashcat -m 22000 hash /opt/wordlist.txt -D 2 --optimized-kernel-enable -w 4 -j '>8'-k '<63'
```

---
# Formats

https://hashcat.net/wiki/doku.php?id=example_hashes

Afficher des informations pour le format de hash attendu par un mode

```SH
hashcat -m 10000 --hash-info
```

Afficher tous les exemples

```sh
hashcat --example-hashes
```

Grep sur un algo

```sh
hashcat --example-hashes | grep -i pbkdf2
```


## PBKDF2

https://notes.benheater.com/books/hash-cracking/page/pbkdf2-hmac-sha256


---
# Crack de fichiers

https://github.com/hashcat/hashcat/tree/master/tools