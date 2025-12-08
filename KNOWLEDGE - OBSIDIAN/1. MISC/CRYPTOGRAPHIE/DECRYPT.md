
- [[#3DES]]
- 


----
# 3DES

Key = 24 octets
IV = 8 octets
String chiffré = IV+string


1. Passer le base64 en hexadecimal afin d'extraire l'IV (les 8 premiers octets).
2. Déchiffrer le reste des octets en utilisant la clé et l'IV.

Script python :

> [!WARNING]
> requiert pycryptodome

```
from Crypto.Cipher import DES3
import base64

def decrypt_3des_cbc(b64_data, key):
    # Decode from Base64
    raw_data = base64.b64decode(b64_data)
    
    # Extract IV (first 8 bytes) and ciphertext
    iv = raw_data[:8]
    ciphertext = raw_data[8:]
    
    # Create cipher object
    cipher = DES3.new(key, DES3.MODE_CBC, iv)
    
    # Decrypt and remove PKCS5/PKCS7 padding
    plaintext = cipher.decrypt(ciphertext)
    pad_len = plaintext[-1]
    plaintext = plaintext[:-pad_len]
    
    return plaintext

if __name__ == "__main__":
    # Example usage
    encrypted_b64 = "L7Rv0xxxxxxxxxxxxxxxxxxxxIk25Am/"
    des_key = b"24-byte-long-des3-key-!!!!!"  # Must be 24 bytes
    
    decrypted = decrypt_3des_cbc(encrypted_b64, des_key)
    print("Decrypted text:", decrypted.decode("utf-8", errors="ignore"))

```

