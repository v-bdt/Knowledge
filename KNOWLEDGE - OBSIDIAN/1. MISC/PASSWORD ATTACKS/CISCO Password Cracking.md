

|**Cisco Password type**|**Ability to crack**|**Vulnerability severity**|**Recommendation**|
|---|---|---|---|
|`Type 0`|Immediate|Critical|Do not use|
|`Type 4`|Easy|Critical|Do not use|
|`Type 5`|Medium|Medium|Use only when Types 6, 8, and 9 are not available|
|`Type 6`|Difficult|Low|Use only when reversible encryption is needed, or when Type 8 is not available|
|`Type 7`|Immediate|Critical|Do not use|
|`Type 8`|Difficult|Low|Recommended|
|`Type 9`|Difficult|Low|Recommended|


# Type 0

Stored plaintext.


# Type 4


```shell-session
john --format=Raw-SHA256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

```shell-session
hashcat -m 5700 -O -a 0 hash /usr/share/wordlists/rockyou.txt
```


# Type 5


```shell-session
john --format=md5crypt --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash
```

```shell-session
hashcat -m 500 -O -a 0 hash /usr/share/wordlists/rockyou.txt
```


# Type 6

Mot de passe chiffré en AES 128. ->
Nécessite la clé de chiffrement AES qui n'est pas stockée dans le fichier de conf.


# Type 7

Basé sur Vigenère cipher dont la clé publique est connue donc possible de déchiffrer.

https://raw.githubusercontent.com/theevilbit/ciscot7/master/ciscot7.py

```shell-session
wget https://raw.githubusercontent.com/theevilbit/ciscot7/master/ciscot7.py
```

```shell-session
python ciscot7.py -d -p 08116C5D1A0E550516
```


# Type 8


```shell-session
john --format=pbkdf2-hmac-sha256 --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash
```

```shell-session
hashcat -m 9200 -a 0 hash /usr/share/wordlists/rockyou.txt
```


# Type 9


```shell-session
john --format=scrypt --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash
```

```shell-session
hashcat -m 9300 -a 0 --force hash /usr/share/wordlists/rockyou.txt
```



# Recommendations


- Eviter Types 0,4,5 et 7
- Utiliser le Type 6 uniquement si besoin de déchiffrer le mot de passe.
- Prioriser le Type 8 selon la NSA et approuvé par le NIST -> https://media.defense.gov/2022/Feb/17/2002940795/-1/-1/1/CSI_CISCO_PASSWORD_TYPES_BEST_PRACTICES_20220217.PDF
- Possible d'utiliser le Type 9 également mais non approuvé par le NIST actuellement. 