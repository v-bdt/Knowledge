
1. Lancer proxy socks sur le port 1080 (socks4a par défaut)

```
socks 1080
```


## Windows

2. Lancer Proxifier (Windows) -> Profile -> Proxy Servers

3. Définir l'ip du serveur C2 et le port 1080

![[Pasted image 20250915210900.png]]

4. Créer une proxification rule dans advanced, laisser Any applications et définir le subnet de la target

![[Pasted image 20250915211029.png]]

5. Possible maintenant de lancer les tools comme ADExplorer, Active Directory ou PowerView en local, le trafic sera redirigé (Penser à créer un PSCredential Object pour les scripts powershell). Ou injecter les tickets kerberos en mémoire avec rubeus ou mimikatz dans une session runas /netonly.


## Linux

1. Utiliser Proxychains.
2. Dans la conf proxychains, commenter la ligne proxy_dns
3. Puis utiliser les tools classiques impacket / netexec...