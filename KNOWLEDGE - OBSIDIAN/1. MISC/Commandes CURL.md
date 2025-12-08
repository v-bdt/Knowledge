

# #GET **REQUESTS**

### *GET request pour récupérer la version du serveur web (Header de réponse)*

```bash
curl <url> --head
```

```bash
curl <url> -I
```

### *GET request en specifiant un utilisateur et un password*

```bash
curl -u admin:admin <url>
```

### *GET request en specifiant un #Cookie de session*

```bash
curl -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' <url>
```

```bash
curl -H 'Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' <url>
```

---

# #POST **REQUESTS**

### *POST request login-form*

```bash
curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/
```

### *POST request avec cookie*

```bash
curl 'http://94.237.54.42:44492/search.php' -X POST -H 'Content-Type: application/json' -H 'Cookie: PHPSESSID=h589kdcps0up9sdh13js3nneqh' -d '{"paramètre":"value"}'
```


---
# #API **REQUESTS**

## #CRUD

### **C**reate -> POST
### **R**ead -> GET
### **U**pdate -> PUT
### **D**elete -> DELETE


### *Creation de data au travers d'une API*

```BASH
curl -X POST 'http://94.237.59.180:48912/api.php/' -d '{"city_name":"New_HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json' 
```

### *Lecture de data au travers d'une API*

```bash
curl -s 'http://94.237.59.180:48912/api.php/city/New_HTB_City' | jq
```

### *Update de data au travers d'une API*

```BASH
curl -X PUT 'http://94.237.59.180:48912/api.php/city/New_HTB_City' -d '{"city_name":"flag", "country_name":"HTB"}' -H 'Content-Type: application/json' 
```

### *Suppression de data au travers d'une API*

```BASH
curl -X DELETE 'http://94.237.59.180:48912/api.php/city/flag'
```