
# BASH

## commandes système

#### Afficher le current path

```powershell
pwd
```

#### Lister les fichiers

```powershell
ls -la <path>
```

#### Lire un fichier

```powershell
cat <fichier>
```


### PARTAGES

#### Monter un partage SMB

> [!important]
> Besoin de CIFS util : `sudo apt install cifs-utils`

avec creds

```shell
sudo mkdir /mnt/share
sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

avec fichier de creds

```txt
username=plaintext
password=Password123
domain=.
```

```shell
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```


#### Rechercher les fichiers contenant un string dans le nom

```shell
find /mnt/Finance/ -name *cred*
```


#### Rechercher un string dans les fichiers

```shell
grep -rn /mnt/Finance/ -ie cred
```

```shell
egrep -ri "cred" /mnt/Finance/
```


