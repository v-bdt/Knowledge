

# OSPF Authentification (HELLO PACKET)

https://www.ietf.org/rfc/rfc2328.txt


### Format john bruteforce

```
OSPF-224.0.0.5-0:$netmd5$OSPF Header(Hex)OSPF Hello Packet(Hex)$Auth Crypt Data
```

### Possible de le mettre au bon format directement avec Ettercap

```bash
ettercap -Tqr capture.pcap 
```

