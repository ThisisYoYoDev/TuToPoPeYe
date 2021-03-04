# TuToPoPeYe

# Welcome to TuTo PoPeYe By YoYoDev!

*Le but de ce README est de vous aider au bon développement de votre projet Popeye de tek1.*

# Architecture
En rouge nous voyons les fichiers les plus importants de votre directory.

![Architecture de notre directory après compilations](https://i.imgur.com/fpYEfAr.png)

## "POLL" Part

Notre premier partir consiste à mettre en place toute la partie "POLL" du projet Popeye.

Dockerfile

```Dockerfile
FROM python:3.8-slim-buster # ICI on install une version de python toute version de python3 marches

COPY . . # ICI on copie le dossier actuel pour le container.
		 
RUN pip3 install -r requirements.txt # ICI cette commande permet d'installer les dépendances python3

EXPOSE 80 # ICI cette command permet d'exposer un port dans le container en l'occurrence le port 80

CMD ["flask", "run", "--host=0.0.0.0", "--port=80"] # ICI on lance tous simplement l'appli FLASK
```

## "RESULT" Part

Notre seconde partir consiste à mettre en place toute la partie "RESULT" du projet Popeye.

```Dockerfile
FROM node:12-alpine

COPY . .

RUN npm install

EXPOSE 80

CMD ["node", "server.js"]
```

## "WORKER" Part

Notre troisième partir consiste à mettre en place toute la partie "RESULT" du projet Popeye.

```Dockerfile
# First stage - compilation
FROM maven:3.5-jdk-8-alpine as builder # La commande "as" permet de le nommer builder dans notre cas

WORKDIR /app # le WORKDIR permet de donner un directory à notre lieu de travail pour ne pas être perdu

COPY . .

RUN mvn dependency:resolve

RUN mvn package

# Second stage - run
FROM openjdk:8-jre-alpine

COPY --from=builder /app/target/worker-jar-with-dependencies.jar . # RTFM

CMD ["java", "-jar", "worker-jar-with-dependencies.jar"]
```

# DOCKER-COMPOSE.YML

Voici un exemple de docker compose fonctionnelle pour notre architecture

```yml
version: "3"

services:
# Service poll
  poll:
    build: ./poll # la commande build permet de lancer le Dockerfile du directory poll 
    restart: always # la commande restart avec l'option always permet de redémarrer le container dès qu'il crash
    ports:
      - 5000:80 # ce sont les ports de connexion RTFM
    networks:
      - poll-tier # le reseaux RTFM

# Service redis
  redis:
    image: redis # l'image est celle d'installation pour le container
    restart: always
    ports:
      - 6379:6379
    networks:
      - poll-tier
      - back-tier

# service worker
  worker:
    build: ./worker
    restart: always
    networks:
      - back-tier

# service db
  db:
    image: postgres
    restart: always
    environment:
      - POSTGRES_PASSWORD=password # RTFM
    networks:
      - result-tier
      - back-tier
    volumes: # RTFM
      - db-data:/var/lib/postgresql/data
      - ./schema.sql:/docker-entrypoint-initdb.d/init.sql

# service result
  result:
    build: ./result
    restart: always
    ports:
      - 5001:80
    networks:
      - result-tier

networks:
  poll-tier:
  result-tier:
  back-tier:

volumes:
  db-data:
```

## Exemple de Résultas
"http://0.0.0.0:5000":



![REQUETE](https://cdn.discordapp.com/attachments/811361338324418643/817099919366160404/unknown.png)









"http://0.0.0.0:5001"
![RESULT](https://cdn.discordapp.com/attachments/811361338324418643/817099993478856734/unknown.png)






Donc la requete a bien été effectue voila vous avez fini le projet Popeye.


Et n'oubliez pas RTFM. ;)

By Yoel :two_hearts:
