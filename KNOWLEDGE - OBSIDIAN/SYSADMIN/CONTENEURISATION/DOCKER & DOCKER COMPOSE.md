
https://docs.docker.com/get-started/docker_cheatsheet.pdf
https://devopscycle.com/blog/the-ultimate-docker-cheat-sheet#how-do-you-access-a-running-container

- [[#DOCKER]]
- [[#DOCKER-COMPOSE]]

# DOCKER

[[#Dockerfile]]
[[#Gestion des images]]
[[#Gestion des containers]]
[[#Gestion des volumes]]

[[#Backup & Restore MYSQL db]]
[[#Automatiser le demarrage de docks]]



## Dockerfile

Permet de construire une image Docker personnalisée grâce à une série d'instructions.

exemple de docker file.

```
# Ceci est un commentaire

FROM docker.io/centos:lastest # Image utilisée pour build la nouvelle image.

MAINTAINER <Author> # Entité en charge de la gestion de l'image.

RUN yum update && yum install -y nginx # Commandes à executer lors du build.

COPY nginx.conf /etc/nginx/nginx.conf # Permet de copier une élément depuis l'hote vers l'image.

RUN chmod 744 /etc/nginx/nginx.conf 

EXPOSE 80 # Expose le conteneur en écoute sur le port indiqué à son demarrage.

ENTRYPOINT [ "ping", "-4" ] # Determine la commande lancée au démarrage d'un conteneur, la gestion des arguments est separée par des virgules. Si non declaré -> /bin/sh -c bash par défaut.

CMD [ "127.0.0.1" ] # Information à passer à la directive ENTRYPOINT si necessaire.

WORKDIR /home/docker # Dossier dans lequel je serais au demarrage du conteneur.
```


## Gestion des images

### Build

Build une image a partir d'un Dockerfile (le tag sert à nommer l'image).

```
docker build . --tag examplename/examplerepository-server:0.1.0
```

Build une image en spécifiant un Dockerfile.

```
docker build --file Dockerfile.client . --tag examplename/examplerepository-server:0.1.0
```

### List

```
docker image ls
```


## Gestion des containers

### Run

Lancer un conteneur a partir d'une image locale.

```
docker run examplename/examplerepository-server:0.1.0
```

Lancer un conteneur depuis un repo.

```
docker run https://registrydomain.com/examplename/examplerepository-server:0.1.0
```

Lancer conteneur en mode détaché (en background)

```
docker run --detached examplename/examplerepository-server:0.1.0
```

Lancer conteneur et exposer le port du conteneur sur un port système

```
docker run --detached --publish 3000:3000 examplename/examplerepository-server:0.1.0
```

### List

Lister les conteneur lancés

```
docker ps
```

```
docker container ls
```

Lister tous les conteneurs

```
docker ps -a
```

```
docker container ls --all
```


### Start / Stop & Remove

lancer conteneur

```
docker container start <container-id>
```

relancer conteneur

```
docker container restart <container-id>
```

arrêter un conteneur

```
docker container stop <container-id>
```

Supprimer conteneur

```
docker container rm <container-id>
```

Kill un conteneur récalcitrant

```
docker kill <container-id>
```


### Execute commands

exécuter une commande à l'intérieur du conteneur

```
docker exec -it <container-id> <shell-command>
```

avoir un shell interactif

```
docker exec -it <container-id> sh
```


## Gestion des volumes

### Run

Lancer conteneur avec un volume nommé

```
docker run --volume <volume-name>:/path/in/container <image-name>
```

Lancer conteneur avec un volume monté

```
docker run --volume /path/on/host:/path/in/container <image-name>
```

### List

lister les volumes

```
docker volume ls
```

## Gestion des réseaux

Les réseaux sous docker sont gérés par docker lui même qui créer des règles iptables.


Créer un réseau

```
docker network create --attachable <nom_du_réseau>
```

Lister les réseaux

```
docker network ls
```

Run un conteneur avec un réseau

```
docker run -d --network <nom_du_réseau> --name <nom_du_dock> -p <port_container>:<port_host>
```


## Backup & Restore MYSQL db

Backup

```
docker exec <CONTAINER> /usr/bin/mysqldump -u root --password=root <DATABASE> > backup.sql
```

Restore

```
cat backup.sql | docker exec -i <CONTAINER> /usr/bin/mysql -u root --password=root <DATABASE>
```


## Automatiser le demarrage de docks

2 solutions possibles:
- Créer des services correspondants aux conteneurs à démarrer
- Via docker lui-même.

### Création de services

1. Créer fichier de configuration dock-"name".service dans /etc/systemd/system/

```
[Unit]
Description= Frontend Registry Hub
Requires=docker.service
After=docker.service Wants=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker run --name frontendHub -e ENV_DOCKER_REGISTRY_HOST=172.31.66.6 -e ENV_DOCKER_REGISTRY_PORT=5000 -p 8080:80 konradkleine/docker-registry-frontend:v2
ExecStop=/usr/bin/docker stop -t 2 frontendHub ; /usr/bin/docker rm -f frontendHub

[Install]
WantedBy=default.target
```

2. Reload la conf systemd pour prendre en compte le fichier crée

```
systemctl daemon-reload
```

3. Enable le service pour qu'il se lance automatiquement au démarrage

```
systemctl enable dock-frontendhub.service
```

### Via docker

Préciser l'option -restart always lors du run du conteneur afin qu'il se lance automatiquement au démarrage du service docker. 

```
docker run -d -restart always -p 25565:25565 –name minecraft penthium/minecraft
```

---

# DOCKER-COMPOSE

