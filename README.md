# Holberton School - Softy Pinko Docker

Projet de containerisation d'une application web avec Docker et Nginx.
L'application est composee d'un front-end statique, d'une API back-end et d'un proxy inverse,
le tout orchestre avec Docker Compose.

---

## Sommaire

- [C'est quoi Docker ?](#cest-quoi-docker-)
- [Image vs Conteneur](#image-vs-conteneur)
- [Dockerfile](#dockerfile)
- [Docker Hub](#docker-hub)
- [Docker Compose](#docker-compose)
- [Commandes essentielles](#commandes-essentielles)
- [Architecture du projet](#architecture-du-projet)
- [Description des taches](#description-des-taches)
- [Lancer le projet](#lancer-le-projet)
- [Load Balancing](#load-balancing)

---

## C'est quoi Docker ?

Avant Docker, le probleme classique etait : "ca marche sur ma machine mais pas sur la tienne".
Les differences de systeme d'exploitation, de versions de librairies ou de variables
d'environnement rendaient le deploiement d'applications complexe et fragile.

Docker resout ce probleme en empaquetant une application avec tout ce dont elle a besoin
(code, dependances, configuration) dans une unite isolee et portable appellee **conteneur**.

Un conteneur tourne de maniere isolee du systeme hote. Il a son propre systeme de fichiers,
ses propres processus et son propre reseau. Pourtant, il partage le noyau du systeme
d'exploitation avec l'hote, ce qui le rend beaucoup plus leger qu'une machine virtuelle.

```
Machine Virtuelle                    Conteneur Docker
+--------------------+               +--------------------+
|   Application      |               |   Application      |
+--------------------+               +--------------------+
|   OS complet       |               |   Librairies       |
|   (plusieurs Go)   |               +--------------------+
+--------------------+               |   Docker Engine    |
|   Hyperviseur      |               +--------------------+
+--------------------+               |   OS Hote          |
|   OS Hote          |               +--------------------+
+--------------------+
```

Les conteneurs demarrent en quelques secondes et consomment tres peu de ressources
comparativement aux machines virtuelles traditionnelles.

---

## Image vs Conteneur

Ces deux notions sont au coeur de Docker et sont souvent confondues.

**L'image** est un modele en lecture seule. Elle contient toutes les instructions
pour construire l'environnement de l'application. Une image ne s'execute pas,
elle sert de base pour creer des conteneurs.

**Le conteneur** est une instance en cours d'execution d'une image. On peut creer
autant de conteneurs qu'on veut a partir d'une seule image.

```
Image Docker
(modele statique)
      |
      |---> Conteneur 1 (en cours d'execution)
      |---> Conteneur 2 (en cours d'execution)
      |---> Conteneur 3 (en cours d'execution)
```

Analogie : l'image est comme un moule a gateau, le conteneur est le gateau.
On peut faire autant de gateaux qu'on veut avec le meme moule.

---

## Dockerfile

Le Dockerfile est le fichier de recette qui decrit comment construire une image.
Chaque instruction cree une nouvelle couche dans l'image.

```dockerfile
# Image de base
FROM ubuntu:latest

# Mise a jour du systeme
RUN apt-get update && apt-get upgrade -y

# Installation de Python
RUN apt-get install -y python3 python3-pip

# Definition du repertoire de travail
WORKDIR /app

# Copie du code source
COPY api.py /app/api.py

# Commande executee au demarrage du conteneur
CMD ["python3", "api.py"]
```

Les instructions les plus courantes :

| Instruction | Role                                              |
|-------------|---------------------------------------------------|
| FROM        | Definit l'image de base                           |
| RUN         | Execute une commande pendant le build             |
| COPY        | Copie des fichiers dans l'image                   |
| WORKDIR     | Definit le repertoire de travail                  |
| EXPOSE      | Documente le port utilise par le conteneur        |
| CMD         | Commande executee au demarrage du conteneur       |
| ENV         | Definit une variable d'environnement              |

Chaque instruction `RUN`, `COPY` ou `ADD` cree une nouvelle couche. Docker met
en cache ces couches pour accelerer les builds suivants. Si une couche n'a pas
change, Docker la reutilise directement sans la reconstruire.

---

## Docker Hub

Docker Hub est le registre public officiel d'images Docker. C'est une sorte de
GitHub pour les images Docker. On peut y trouver des images officielles maintenues
par les editeurs de logiciels (nginx, ubuntu, python, mysql...) ainsi que des images
publiees par la communaute.

Quand on ecrit `FROM nginx:latest` dans un Dockerfile, Docker telecharge
automatiquement l'image `nginx` depuis Docker Hub si elle n'est pas deja
presente en local.

Les images ont des tags qui designent une version precise :
- `nginx:latest` - la derniere version stable
- `ubuntu:22.04` - Ubuntu version 22.04
- `python:3.11-slim` - Python 3.11 en version allégée

---

## Docker Compose

Docker Compose est un outil qui permet de definir et gerer des applications
composees de plusieurs conteneurs. On decrit l'ensemble de l'application dans
un fichier `docker-compose.yml`.

```yaml
services:
  back-end:
    build:
      context: ./back-end
      dockerfile: dockerfile
    image: mon-back-end:v1

  front-end:
    build:
      context: ./front-end
      dockerfile: dockerfile
    image: mon-front-end:v1
    depends_on:
      - back-end

  proxy:
    build:
      context: ./proxy
      dockerfile: dockerfile
    ports:
      - "80:80"
    depends_on:
      - front-end
      - back-end
```

Les concepts cles de Docker Compose :

**services** : chaque service correspond a un conteneur. On peut en definir autant
que necessaire dans le meme fichier.

**build** : indique ou trouver le Dockerfile pour construire l'image du service.

**image** : nom donne a l'image une fois construite.

**ports** : mappe un port du conteneur sur un port de la machine hote.
Le format est `"port_hote:port_conteneur"`. Sans cette instruction, le port
du conteneur n'est pas accessible depuis l'exterieur.

**depends_on** : definit l'ordre de demarrage des services. Un service avec
`depends_on: - back-end` ne demarrera qu'apres le service `back-end`.

**reseau interne** : Docker Compose cree automatiquement un reseau prive entre
tous les services. Les services peuvent communiquer entre eux en utilisant
directement leur nom de service comme nom d'hote (ex: `http://back-end:5252`).

---

## Commandes essentielles

### Construction et demarrage

```bash
# Construire les images et demarrer les conteneurs
docker-compose up --build

# Demarrer en arriere-plan
docker-compose up --build -d

# Demarrer avec plusieurs instances d'un service
docker-compose up --build --scale back-end=3
```

### Arret et nettoyage

```bash
# Arreter les conteneurs
docker-compose down

# Arreter et supprimer les volumes
docker-compose down -v

# Arreter et supprimer les images
docker-compose down --rmi all
```

### Inspection

```bash
# Lister les conteneurs en cours d'execution
docker-compose ps

# Afficher les logs en temps reel
docker-compose logs -f

# Afficher les logs d'un service specifique
docker-compose logs -f proxy

# Entrer dans un conteneur en cours d'execution
docker exec -it nom_du_conteneur bash
```

### Gestion des images

```bash
# Lister les images locales
docker images

# Supprimer une image
docker rmi nom_image

# Supprimer toutes les images non utilisees
docker image prune
```

---

## Architecture du projet

```
Navigateur
    |
    v
[Proxy Nginx :80]   <-- seul port expose sur la machine hote
    |            |
    v            v
[Front-end]  [Back-end(s)]
  :9000          :5252
  (Nginx)        (Flask)
  statique       dynamique
```

Le proxy Nginx est le point d'entree unique. Il analyse l'URL de chaque requete
et la redirige vers le bon service :

- `http://localhost/`       -> front-end (page HTML, CSS, JS)
- `http://localhost/api`    -> back-end (reponse JSON de l'API)

Les services front-end et back-end ne sont pas directement accessibles depuis
l'exterieur. Seul le proxy expose le port 80.

---

## Description des taches

### Task 0 - Image de base Ubuntu

Creation d'une image Docker a partir d'Ubuntu avec les mises a jour systeme.

```
task0/
  dockerfile
```

### Task 1 - Serveur back-end

Ajout d'une API Python Flask qui repond sur le port 5252.

```
task1/
  dockerfile
  api.py
```

### Task 2 - Serveur front-end

Ajout d'un serveur Nginx qui sert le contenu statique du site Softy Pinko.

```
task2/
  back-end/
  front-end/
```

### Task 3 - Connexion front-end / back-end

Le front-end appelle l'API back-end pour recuperer des donnees dynamiques.

```
task3/
  back-end/
  front-end/
```

### Task 4 - Docker Compose

Orchestration des deux services avec un fichier `docker-compose.yml`.

```
task4/
  docker-compose.yml
  back-end/
  front-end/
```

### Task 5 - Proxy inverse

Ajout d'un proxy Nginx qui centralise les requetes et les redistribue
vers le bon service selon l'URL.

```
task5/
  docker-compose.yml
  proxy/
  back-end/
  front-end/
```

### Task 6 - Load Balancing

Deux serveurs back-end tournent en parallele. Le proxy Nginx repartit
les requetes entre eux selon l'algorithme Round-Robin.

```
task6/
  docker-compose.yml
  proxy/
  back-end/
  front-end/
  2-api-servers.txt
```

---

## Lancer le projet

Chaque tache est independante. Pour lancer une tache specifique :

```bash
cd taskX
docker-compose up --build
```

Remplacer `X` par le numero de la tache souhaitee (4, 5 ou 6).

L'application est accessible sur : `http://localhost`

Pour arreter les conteneurs :

```bash
docker-compose down
```

---

## Load Balancing

La tache 6 met en place un load balancer avec deux serveurs back-end.
Le simple `docker-compose up --build` suffit a demarrer les deux serveurs.

### Pourquoi le load balancing ?

Quand le trafic augmente, un seul serveur peut etre sature et ne plus repondre
assez vite. Le load balancing consiste a repartir les requetes sur plusieurs
serveurs pour distribuer la charge.

### Round-Robin

Le Round-Robin est l'algorithme de load balancing le plus simple. Chaque nouvelle
requete est envoyee au serveur suivant dans la liste, de facon cyclique.

```
Requete 1  ->  back-end-1
Requete 2  ->  back-end-2
Requete 3  ->  back-end-1
Requete 4  ->  back-end-2
...
```

Chaque serveur recoit exactement le meme nombre de requetes. C'est le comportement
par defaut du bloc `upstream` dans Nginx.

### Configuration Nginx

```nginx
upstream back-end {
    server back-end-1:5252;
    server back-end-2:5252;
}

server {
    listen 80;

    location /api {
        proxy_pass http://back-end;
    }
}
```

Le bloc `upstream` definit un groupe de serveurs. Nginx cycle a travers la liste
et envoie chaque requete au serveur suivant. Les serveurs sont identifies par leur
nom de service Docker et leur port interne.

### Autres algorithmes disponibles dans Nginx

```nginx
upstream back-end {
    least_conn;              # envoie vers le serveur le moins charge
    server back-end-1:5252;
    server back-end-2:5252;
}
```

```nginx
upstream back-end {
    ip_hash;                 # un meme client va toujours vers le meme serveur
    server back-end-1:5252;
    server back-end-2:5252;
}
```

---

## Auteur

Hinoto-LH - Holberton School
