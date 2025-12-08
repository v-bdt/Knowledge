

# TLS (TCP)


TLS repose sur deux types de chiffrement :

1. Chiffrement asymétrique (au début de la connexion)
2. Chiffrement symétrique (pour le trafic après l’établissement de la session)

Process:

1. TCP 3way handshake
2. Chiffrement asymétrique pour établir la session (échange de certificats)
	1. Le serveur envoie son certificat au client
	2. Le client vérifie la validité du certificat
	3. Le serveur envoie sa clé publique (RSA par ex)
	4. Le client génère une clé de session qu'il va chiffrer à l'aide de la clé publique RSA et l'envoie au serveur.
	5. Le serveur reçoit la clé de session et la déchiffre à l'aide de la clé RSA privé.
3. Chiffrement symétrique (pour le trafic après l’établissement de la session)
	1. La clé de session est ensuite utilisée par le client et le serveur pour chiffrer et déchiffrer les données échangées.

![[Pasted image 20250321185706.png]]