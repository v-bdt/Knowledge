

# Do's

- Log toutes ses actions
- Comprendre les conséquences de l'utilisation d'un outil (Les changements effectués sur la cible, les artefacts laissés)
	- Toujours rétablir les changements effectués par un tool.
	- Nettoyer les artefacts laissés afin de ne pas se faire attraper par la Blue Team.
- "Be aware" -> Situational Awareness afin de s'assurer d'être dans le scope défini, les protections existantes etc...

# Dont's

- Utiliser des tools pré-compilés
	- Toujours investiguer le comportement d'un tool afin de s'assurer que celui-ci ne contienne pas de backdoor ou ne commette pas des choses destructrices, puis le compiler sois même.
- Utiliser des canaux de C2 non chiffrés
	- Toujours s'assurer que les canaux utilisés sont chiffrés, oublier les tools comme netcat etc qui envoie les données en clair sur le réseau (et donc vulnérable au sniffing).
- Exfiltrer des données sensibles
	- A moins que spécifiquement demandé par le client, ne jamais exfiltrer des données sensibles telles que des numéros de carte de crédits, données sensibles, données commerciales, personnelles etc...
	- Si besoin de tester la DLP, créer un fichier factice qui mimique les données que le client souhaite protéger.
- Désactiver les sécurités en place
	- Ne jamais désactiver les sécurités en place (AV / Firewall) afin de ne pas faire courir de risque au client.