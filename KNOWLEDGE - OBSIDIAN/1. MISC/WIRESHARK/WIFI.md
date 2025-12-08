
1. `Beacon Frames`
2. `Probe Request and Response`
3. `Authentication Request and Response`
4. `Association Request and Response`
5. `Some form of handshake or other security mechanism`
6. `Disassociation/Deauthentication`



## FIltres

1. Beacon Frames

```
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 8)
```

2. Probe Requests

```
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 4)
```

3. Probe Responses

```
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 5)
```

4. Authentication Process

```
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 11)
```

5. Association Request

```
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 0)
```

6. Association Response

```
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 1)
```

7. WPA2 EAPOL

```
eapol
```

8. Deauth / Disas

```
`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 12) or (wlan.fc.type_subtype == 10)`
```