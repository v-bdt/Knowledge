
1. Algos de chiffrement symétrique
	1. Par Bloc (Blocs de taille fixe)
		- **AES (Advanced Encryption Standard)** – 128, 192, ou 256 bits (standard actuel)
		- **DES (Data Encryption Standard)** – 56 bits (obsolète, vulnérable aux attaques)
		- **3DES (Triple DES)** – 168 bits (plus sécurisé que DES mais dépassé)
		- **Blowfish** – 32 à 448 bits (rapide mais remplacé par AES)
		- **Twofish** – 128, 192 ou 256 bits (successeur de Blowfish)
		- **Camellia** – 128, 192, ou 256 bits (équivalent d'AES, utilisé au Japon)
		- **Serpent** – 128, 192, ou 256 bits (finaliste d'AES, très sécurisé mais lent)
		- **IDEA (International Data Encryption Algorithm)** – 128 bits (peu utilisé aujourd'hui)
		
	2. Par flux (bit par bit)
		- **RC4 (Rivest Cipher 4)** – rapide mais vulnérable (déconseillé)
		- **ChaCha20** – 256 bits (plus sécurisé et plus rapide qu’AES sur CPU sans accélération matérielle)
		- **Salsa20** – 256 bits (ancêtre de ChaCha20)
	
2. Algos de chiffrement asymétrique
	1. Logarithmes modulo
		- **RSA (Rivest-Shamir-Adleman)** – 1024, 2048, 4096 bits (standard mais lent)
		- **Rabin** – variante de RSA, basé sur la factorisation
		- **Paillier** – cryptosystème homomorphe partiel (utilisé en cryptographie avancée)
		
	2. Courbes elliptiques (ECC -> Plus rapide et sécurisé que RSA à taille de clé équivalente)
		- **ECDSA (Elliptic Curve Digital Signature Algorithm)** – pour les signatures
		- **ECDH (Elliptic Curve Diffie-Hellman)** – pour l’échange de clés
		- **Ed25519** – alternative plus rapide et sécurisée d’ECDSA
		- **Curve25519** – alternative d’ECDH (utilisé par Signal, WireGuard)
		- **SM2** – algorithme ECC chinois
		
	3. Diffie-Hellman
		- **Diffie-Hellman (DH)** – 2048 bits minimum recommandé
		- **ECDH (Elliptic Curve Diffie-Hellman)** – variante basée sur ECC (plus rapide)
		- **X25519** – variante optimisée d’ECDH (utilisé par TLS 1.3)
		
	4. Réseaux euclidiens (Cryptographie Post-Quantique)
		- **NTRU** – résiste aux ordinateurs quantiques
		- **Kyber** – candidat pour le standard post-quantique
		- **SIDH/SIKE** – basé sur les isogénies de courbes elliptiques (cassé en 2022)

