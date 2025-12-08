


# ENUMERATION

## Tilde Directory

> [!TIP]
> Uncover hidden files, directories, and short file names (aka the `8.3 format`).
> The tilde (`~`) character, followed by a sequence number, signifies a short file name in a URL.
> **Fuzzing caractère par caractère** (~a -> ~ar -> ~ari ...) (Status 200 OK pour chaque requete)

For example, if two files named `somefile.txt` and `somefile1.txt` exist in the same directory, their 8.3 short file names would be:

- `somefi~1.txt` for `somefile.txt`
- `somefi~2.txt` for `somefile1.txt`



https://github.com/irsdl/IIS-ShortName-Scanner

```bash
cd release
```

```shell
java -jar iis_shortname_scanner.jar 2 5 http://10.129.179.159/
```

Si fichier trouvé mais non accessible -> Bruteforce du nom de fichier complet

Générer wordlist a partir du nom de fichier identifié (TRANSF~1.ASP)

```shell
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```

Eumération avec GoBuster

```shell
gobuster dir -u http://10.129.179.159/ -w /tmp/list.txt -x .aspx,.asp
```

