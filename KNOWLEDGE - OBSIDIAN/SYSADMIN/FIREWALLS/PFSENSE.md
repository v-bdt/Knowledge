
# Download

https://atxfiles.netgate.com/mirror/downloads/


# Config VM

2GB de RAM minimum
1 Proc 2 cœurs minimum
SCSI -> LSI
20GB d'espace Disk

Ajouter au total 3 cartes réseau
Créer 3 segments réseau Admin, LAN et DMZ

Mettre une carte en bridge et les 2 autres dans leur segment respectif.

Lancer la VM

# Installation du PF

1. Accepter les conditions d'utilisation
2. Sélectionner Install PFSense
3. Utiliser le mode "**Auto (ZFS)**"
4. Valider l'install avec entrée
5. Sélectionner le mode de redondance souhaité
6. Sélectionner le disque virtuel en appuyant sur espace puis entrée et sélectionner Yes
7. Valider le Reboot

# Configuration du PF


## Interfaces

1. Sélectionner l'option 2
2. Sélectionner l'interface LAN avec l'option 2
3. Ne pas configurer via DHCP
4. Attribuer une adresse IP statique
5. Choisir un Masque de sous réseau
6. Pas de passerelle
7. Pas d'IPv6
8. Pas de DHCP

# Panneau d'admin

### default creds

admin:pfsense

## 1. Wizard Setup

1. Aller dans system -> Setup Wizard
2. Next
3. Next
4. Attribuer les DNS
5. Choisir une timezone
6. Décocher les 2 options suivantes : "**Block private networks form entering via WAN**" et "**Block non-internet routed networks from entering via WAN**"
7. Next
8. Changer le mot de passe admin
9. Reload
10. Finish

## 2. Ajouter une interface DMZ

1. Aller dans Interfaces -> Assignements
2. Cliquer sur Add pour ajouter l'interface em2
3. Save
4. Cliquer sur l'interface
5. L'activer, la renommer et attribuer une Ipv4 statique et un masque sur un réseau différent du LAN.
6. Save
7. Apply changes

## 3. Ajouter une interface Administration

1. Aller dans Interfaces -> Assignements
2. Cliquer sur Add pour ajouter l'interface em3
3. Save
4. Cliquer sur l'interface
5. L'activer, la renommer et attribuer une Ipv4 statique et un masque sur un réseau différent du LAN.
6. Save
7. Apply changes


## 4. Règles de Firewall


1. Aller dans Firewall -> Rules


### LAN - Bloquer les flux vers la DMZ

1. Sélectionner l'interface LAN et cliquer sur Add
	1. Action: Block
	2. Interface: LAN
	3. Address Family: IPv4+IPv6
	4. Protocol: Any
	5. Source: LAN subnets
	6. Destination: DMZ subnets
	7. Description: Bloquer les flux LAN vers DMZ
	8. Save

### LAN - Bloquer les flux vers l'Administration

1. Sélectionner l'interface LAN et cliquer sur Add
	1. Action: Block
	2. Interface: LAN
	3. Address Family: IPv4+IPv6
	4. Protocol: Any
	5. Source: LAN subnets
	6. Destination: Administration subnets
	7. Description: Bloquer les flux LAN vers Administration
	8. Save

### DMZ - Bloquer les flux vers le LAN

1. Sélectionner l'interface DMZ et cliquer sur Add
	1. Action: Block
	2. Interface: DMZ
	3. Address Family: IPv4+IPv6
	4. Protocol: Any
	5. Source: DMZ subnets
	6. Destination: LAN subnets
	7. Description: Bloquer les flux DMZ vers LAN
	8. Save
### DMZ - Bloquer les flux vers l'Administration

1. Sélectionner l'interface DMZ et cliquer sur Add
	1. Action: Block
	2. Interface: DMZ
	3. Address Family: IPv4+IPv6
	4. Protocol: Any
	5. Source: DMZ subnets
	6. Destination: Administration subnets
	7. Description: Bloquer les flux DMZ vers Administration
	8. Save


# VPN


## Client-à-site OpenVPN



