

Collecteur vCenterHound pour collecter les données de vCenter. 
https://github.com/MorDavid/vCenterHound



Collecter les données sur un vCenter

```sh
python3 vCenterHound.py --server vc.example.com -u admin@vsphere.local -p "Password!" --port 443 -o custom_graph.json
```

Collecter les données sur de multiples vCenter

```sh
python3 vCenterHound.py --server vc.example1.com,vc.example2.com,vc.example3.com -u admin@vsphere.local -p "Password!" --port 443 -o custom_graph.json
```


Puis upload le fichier json sur BloodHound