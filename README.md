# Desplegar App Contenerizada en un Clúster Swarm

En esta guía vamos a desplegar una Aplicación desarrollada con Nodejs que utiliza una base de datos
NoSQL MongoDB en un clúster Swarm. Para monitorizar los ccontenedores usaremos la aplicación visualizer
en un contenedor.


## Crear el clúster

Crear 3 máquinas virtuales, una de ellas la denominaremos: master y las otras dos: nodo1 y nodo2.

Instalamos docker en cada una de ellas.
Instalamos docker-compose en el máster

Iniciamos el Swarm en la máquina máster
```js
$ docker swarm init

Swarm initialized: current node (sqab0b4s7an7rbnxfujwqzsjq) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-2-2755s1w5d5ssynkmlz1qvrorw6lox4p5f0ibjgp4mh1p038t6d-ax1xe0bdhal9pg8a8zbkvf736 10.132.0.10:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```

Unimos al Swarm los otros dos nodos
```js
$ docker swarm join --token SWMTKN-2-2755s1w5d5ssynkmlz1qvrorw6lox4p5f0ibjgp4mh1p038t6d-ax1xe0bdhal9pg8a8zbkvf736 10.132.0.10:2377

This node joined a swarm as a worker.

```

Ver el estado del Swarm:

## Construye la imagen de la aplicación a partir del Dockerfile

```js
docker build -t jluisalvarez/node-webapp .
```

## Docker Compose

El fichero doker-compose.yml tiene definidos 3 servicios, una red para conectarlos y un volumen para persistir los datos

```js
version: "3.7"
services:
  web:
    imagen: jluisalvarez/node-webapp
    depends_on:
      - mongo
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:8080"
    networks:
      - webnet

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet

  mongo:
    image: mongo
    volumes:
      - db-data:/data/db
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet

volumes:
  db-data:

networks:
  webnet:
```


## Desplegar servicios:

```js
docker stack deploy -c docker-compose.yml webapp
```

## Mostrar listado de servicios

```js
docker service ls
```

## Eliminar servicios

```js
docker stack rm webapp
```