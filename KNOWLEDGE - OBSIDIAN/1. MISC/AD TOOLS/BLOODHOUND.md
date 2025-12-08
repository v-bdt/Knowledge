
> [!TIP]
> Permet d'identifier les chemins d'attaque.

> [!attention]
> Une fois l'accès obtenu à un hôte faisant parti du domaine, lancer sharphound depuis celui ci permet d'obtenir bien plus de données qu'avec la collecte en remote avec netexec...


- [[#1. COLLECTE DE DONNEES (Sharphound)]]
- [[#2. ANALYSE DE DONNEES (Bloodhound GUI)]]


- [[#Reset mot de passe admin perdu]]


# 1. COLLECTE DE DONNEES (Sharphound)

> [!WARNING]
> Mauvais d'un point de vue OPSEC, il est préférable de faire des requêtes LDAP et cibler au maximum les requêtes.

Méthodes de collecte possibles:

https://github.com/dirkjanm/BloodHound.py/blob/master/README.md#usage

# LINUX
## *Netexec*

Collecter toutes les données (port 636 si ldaps)

```bash
nxc ldap <ip> -u user -p pass -d <domain> --kdcHost DANTE-DC02 --dns-server 172.16.2.5 --bloodhound --collection All --port 636 --timeout 30 --debug
```


## *Rusthound-ce*

https://github.com/g0h4n/RustHound-CE

> [!Tip]
> Collecte également les données relatives aux CA

```sh
rusthound-ce -d darkzero.htb -u 'john.w' -p 'RFulUtONCOL!' -z --ldaps
```

## *Bloodhound.py*

https://www.kali.org/tools/bloodhound.py/

> [!Warning]
> Compatible avec Bloodhound legacy (<=4.3.1)

ajouter le domain et ns au resolv.conf

```shell
domain DOMAIN.LOCAL
nameserver 10.80.80.2
```

Collecter les données

```bash
bloodhound-python -c All --zip -d '<domain>' -u 'john.doe' -p 'P@$$word123!' -ns 10.80.80.2 --use-ldaps
```

## *Bloodhound-Ce-Python*

https://www.kali.org/tools/bloodhound-ce-python/

Nouvelle version de Bloodhound.py (>4.3.1)

ajouter le domain et ns au resolv.conf

```shell
domain DOMAIN.LOCAL
nameserver 10.80.80.2
```

Collecter les données

```bash
bloodhound-ce-python -c All --zip -d '<domain.local>' -u 'john.doe' -p 'P@$$word123!' -ns 10.80.80.2 --use-ldaps
```


Importer données dans Bloodhound

Une fois les données collectées, les importer dans bloodhound GUI

```bash
sudo neo4j start
```

```bash
bloodhound
```

# WINDOWS

> [!WARNING]
> Mauvais d'un point de vue OPSEC, il est préférable de faire des requêtes LDAP.

https://github.com/SpecterOps/SharpHound/releases/tag/v2.6.3


```powershell
.\SharpHound.exe --help
```

Collecter toutes les données

```powershell
.\SharpHound.exe -c All --zipfilename ILFREIGHT
```

Collecter toutes les données en précisant des credentials

```
.\SharpHound.exe -c All --zipfilename zsm.local --ldapusername jamie --ldappassword P@ssw0rd
```



---

# 2. ANALYSE DE DONNEES (Bloodhound GUI)


> [!ATTENTION]
> Penser a ajouter tous les objets owned (Users/Computers/Groups...)
> Relancer la query "Shortest Path from Owned objects" à chaque nouvel objet owned ajouté

## 2.3. ACLs

Sélectionner un User > Node Info > Outbound Control Rights > Transitive Object Control

## 2.4. Trust Relationships

Analysis > Pre-Built Analytics Queries > Domain Information > Map Domain Trusts

## 2.5. Groupes restreints

Identifier les Admins locaux des computers

https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-ada1/6eb770b7-0c89-4a3e-a41e-2807d46880d8


1. Dans bloodhound, afficher toutes les GPOs grâce aux cypher queries

```cypher
Match (n:GPO) return n
```

2. Identifier le gpcpath de la GPO cible et le noter

![[Pasted image 20250915025103.png]]

3. Cliquer sur Affected Objects -> Computers, puis cliquer sur le computers pour noter leur SID

![[Pasted image 20250915025420.png]]

Identifier le fichier GptTmpl.inf dans le Gpcpath\Machine\Microsoft\Windows NT\SecEdit\

```
ls <gpcpath>\Machine\Microsoft\Windows NT\SecEdit\
```

Télécharger le fichier GptTmpl.inf qui contient les groups membership

```
download <gpcpath>\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf
```

Synchroniser le fichier sur notre machine locale -> View -> Downloads -> Sync

L'ouvrir et noter le SID membre de *S-1-5-32-544.

```
[Unicode]
Unicode=yes
[Version]
signature="$CHICAGO$"
Revision=1
[Group Membership]
*S-1-5-21-3926355307-1661546229-813047887-1107__Memberof = *S-1-5-32-544 > built in admin group
*S-1-5-21-3926355307-1661546229-813047887-1107__Members =
```

Mapper les groupes identifiés comme Administrateur local aux computers affectés par la GPO

```
MATCH (x:Computer{objectid:'S-1-5-21-3926355307-1661546229-813047887-1110'})
MATCH (y:Group{objectid:'S-1-5-21-3926355307-1661546229-813047887-1107'})
MERGE (y)-[:AdminTo]->(x)
```



## 2.5 CYPHER QUERIES

https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/


### Trouver les users qui peuvent PSRemote / RDP

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanRDP*1..]->(c:Computer) RETURN p2
```

### Trouver SQLAdmin

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```


### Find all the Edges that any UNPRIVILEGED user (based on the admincount:False) has against all the nodes:    

```Cypher
MATCH (n:User {admincount:False}) MATCH (m) WHERE NOT m.name = n.name MATCH p=allShortestPaths((n)-[r:MemberOf|HasSession|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|CanRDP|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|ReadGMSAPassword|DCSync|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p
```

idem pour les groupes

```Cypher
MATCH (n:Group {admincount:False}) MATCH (m) WHERE NOT m.name = n.name MATCH p=allShortestPaths((n)-[r:MemberOf|HasSession|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|CanRDP|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|ReadGMSAPassword|DCSync|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p
```

idem pour les Computers

```Cypher
MATCH (n:Computer) MATCH (m) WHERE NOT m.name = n.name MATCH p=allShortestPaths((n)-[r:MemberOf|HasSession|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|CanRDP|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|ReadGMSAPassword|DCSync|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p
```



Afficher toutes les GPOs

```cypher
Match (n:GPO) return n
```



---
# Reset mot de passe admin perdu


1. Se connecter à la base postgresql

```
sudo -u postgres psql
```

2. Switch sur la base bloodhound

```
\c bloodhound
```


3. Identifier le user admin

```
SELECT * FROM users;
```

4. Supprimer tout dans ingest_jobs

```
DELETE FROM ingest_jobs;
DELETE FROM users WHERE principal_name='admin';
```

Si message d'erreur

```
ERROR: update or delete on table "users" violates foreign key constraint "fk_users_roles_user" on table "users_roles" DETAIL: Key (id)=(d9fef873-2e2b-4eb7-b704-26d720cafb01) is still referenced from table "users_roles".
```

5. Supprimer ce qui correspond au user_id dans users_roles.

```
DELETE FROM users_roles WHERE user_id='d9fef873-2e2b-4eb7-b704-26d720cafb01';
```

6. Puis supprimer l'admin

```
DELETE FROM users WHERE principal_name='admin';
```

7. Enfin lancer bloodhound avec variable d'environnement pour recréer le compte admin

```
sudo env bhe_recreate_default_admin=true bloodhound
```

