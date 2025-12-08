
https://github.com/bitsadmin/wesng


1. Récupérer les infos system depuis la cible

```
systeminfo > systeminfo.txt
```

2. Exfiltrer le fichier

3. Mettre à jour la base de CVE de WES

```sh
./wes.py --update
```

4.  Afficher les vulnérabilités avec des exploits publiques

```sh
./wes.py -e systeminfo.txt
```

5.  Afficher toutes les vulnérabilités

```sh
./wes.py systeminfo.txt
```

6. Récupérer les identifiants des CVEs

```sh
./wes.py systeminfo.txt -i "Elevation of Privilege" | grep CVE | cut -d ":" -f 2 | sed 's/ //' | sort -u > CVEs.txt
```

7. Mettre à jour searchsploit

```sh
searchsploit -u
```

8. Rechercher PoC sur exploit-db pour chaque CVE

```sh
for CVE in $(cat CVEs.txt); do echo "$CVE:" >> PoC_CVEs.txt; searchsploit --cve $CVE | sed '/No Results/d' | grep "/" | tee -a PoC_CVEs.txt ; done 
```

9. Télécharger tous les PoC

```sh
mkdir PoC
cd PoC
for PoC in $(grep "/" ../PoC_CVEs.txt | cut -d '|' -f 2 | sed 's/ //'); do searchsploit -m $PoC | tee -a results.txt; done
```
