###### To check docker setup

`sudo docker run hello-world`

###### To check all running and stopped containers

`docker ps -a`

###### To check docker version

`docker --version`

docker version

docker info

docker

docker image --help

docker container ls

docker container ls -a

docker search nginx

docker container run --publish 8080:80 nginx

docker container run --publish 8080:80 --detatch nginx

docker container run --publish 8081:80 --detatch nginx

docker container stop `<id>`

docker container start `<id>`

docker container rm `<name>`

docker container rm -f `<name>`

docker container run --publish 8080:80 --detach --name nig1 nginx

docker container run -p 8080:80 -d --name nig1 nginx

docker container logs nig1

docker container top nig1

ps -ef | grep nginx

docker run --name custom_mysql -e MYSQL_ROOT_PASSWORD=123 -d mysql

docker stats <container_name>

docker inspect <container_name>

docker container run -p 8080:80 -it nig1 nginx /bin/bash

find / -name "index.html"

cat /usr/share/nginx/html/index.html

exit

docker exec -it custom_mysql mysql -uroot -p

exit

# Docker Networking

### Introduction

docker run -p 8080:80 -d --name nig2 nginx

docker run -p 8081:80 -d --name nig3 nginx

docker inspect nig2(and then find ip address) (its ip address is 172.17.0.2)

docker inspect nig3(and then find ip address) (its ip address is 172.17.0.3)

docker exec -it nig2 /bin/bash(this will go into container of nig2)

ping 172.17.0.3(ping nig3 from nig2. this is how we can communicate under defauilt bridge network, ie, through ip addresses)

### Docker container Networks

docker run -p <host_port>:<container_port> .......

docker port <container_name>

docker inspect <container_name>

docker inspect nig2 -f {{.NetworkSettings.IPAddress}}

### CLI Operations

docker network create <container_name>

docker network inspect <container_name>

docker network rm <container_name>

docker network prune

docker netword stop <contianer_name>

### DNS Concept

docker network create dns_bridge

docker run -p 8088:80 -d --name alpine_container --network dns_bridge nginx:alpine

docker run -p 8090:80 -d --name alpine_container_2 --network dns_bridge nginx:alpine

docker exec -it alpine_container ping alpine_container_2

# Docker Container Images: Beginning

### Concept of Docker Image Layers

docker images

docker pull mysql:8.0

docker history <image_name>(will show layers of image)

### Concept Image Tagging

docekr tag nginx:latest nginx_noel:1.0.0 (change tag of an image, by creating a seperate image)

# Docker Container Images_Build Container Images

### Docker file format

FROM <image_name>:<image_tag>

LABEL `<key>="<value>`"

RUN `<commands>`

CMD ["executable", "param1 ", "param2"]

EXPOSE `<port>`

ENV (set environment variables)

ADD (adds/copies files)

COPY (only copies)

VOUME (expose database storage areas)

WORKDIR (set the pwd)

### Creating custom docker image

1. create "dockerfile" in vs code
2. 

FROMubuntu:latest

LABELversion="1.0"

LABELdescription="ThisisasampleDockerfileforbuildingasimpleUbuntu-basedimage."

LABELmaintainer="noelmathen03@gmail.com"

RUNapt-getupdate&&\

    apt-getupgarade-y

RUNapt-getinstallnginx-y

EXPOSE80

CMD["nginx","-g","daemonoff;"]

3. mkdir customimages
4. vi dockerfile
5. press i
6. paste the dockerfile code from vscode to cloud
7. press esc
8. type :wq
9. docker build -t noelmathen/nginx_latest:1.0
10. docker images

### Extending docker official image

1. mkdir extendedImage
2. vi extendedImage/dockerfile
3. cd extendedImage/
4. create dockerfile:

FROMnginx:latest

LABELversion="1.0"

LABELdescription="ThisisasampleDockerfileforbuildingasimpleNginx-basedimage."

WORKDIR/usr/share/nginx/html

COPYindex.htmlindex.html

5. create index.html
   1. `<h1>` Extended docker image `</h1>`
6. docker build -t noelmathen/nginx_extended:1.0 .
7. docker run -it -d -p 8081:80 --name nginx_extended noelmathen/nginx_extended:1.0

# Handle Persistent Data in Docker Containers

### Handle with data Volumes

docker pull mysql:8.2

docker run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:8.2

cd /var/lib/docker/volumes/a7965947970ef7a6716ccbe2f6ac338384b84b1a70132a3406958e9e14c95e70/_data (data in host machine)

docker exec -it mysql \bin\bash

cd var/lib/mysql (data in container)

Note: now even if the container is deleted, the data in the host machinte is nto deleted, thus data is persistent.

docker volume ls

docker volume prune -a

docker volume create `<name>`

docker volume prune `<name>`

docker run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true --mount source=mysql_db,target=/var/lib/mysql  --name mysql2 mysql:8.2(new container is created but uses the same volume of mysql itself)

docker run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true -v mysql_db:/var/lib/mysql  --name mysql3 mysql:8.2 (another way to run with volumes using -v shortcut)

### Handle with data binds

bind mounts can be stored anywhere.

mkdir testbind

docker run -d --mount type=bind,source=/root/testbind,target=/apps -p 8080:80 --name nginx1 nginx:latest

open new terminal, then => docker exec -it nginx1 /bin/bash => cd apps => ls

you can see that the testbind directory made in the host machine is now visible in the container also, in real time, wihtout any redeployment or anything. In the volume bind case, this won't work. If we try to change the volume mounts within the container when more than 1 containers are using the same volume, then it will automatically terminate containers randomly.

# Section 9 - Docker Compose -Multi Container Orchestration

### Docker-compose YAML file structure example:

version: "3.8"  # Defines the syntax version. Always start here.

services:
  web:
    build:
      context: ./web            # Folder where Dockerfile exists
      dockerfile: Dockerfile.dev  # Optional: custom Dockerfile name
      args:
        build_var: "123"        # Build-time ARG (not ENV)
    image: myapp_web:1.0         # Name the built image
    container_name: webapp       # Optional name override
    ports:
      - "8080:80"               # Host:Container port mapping
    environment:
      - NODE_ENV=development
      - API_URL=http://api:5000
    env_file:
      - .env                    # Load env vars from file
    volumes:
      - ./html:/usr/share/nginx/html      # Bind mount
      - app-logs:/var/log/nginx            # Named volume
    depends_on:
      - db                      # Wait for db to be created before starting
    restart: always             # Auto-restart on failure or reboot
    command: ["nginx", "-g", "daemon off;"]  # Override CMD in Dockerfile
    entrypoint: ["sh", "-c"]    # Override ENTRYPOINT
    working_dir: /app           # Set working directory inside container
    extra_hosts:
      - "host.docker.internal:host-gateway"  # For host<->container access
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"] # Health command
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    networks:
      - app-network

  db:
    image: mysql:5.7
    container_name: mysql-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: appdb
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - db
    networks:
      - app-network

volumes:
  db-data:           # Named volume for DB
  app-logs:          # Named volume for NGINX logs

networks:
  app-network:       # Custom bridge network

### Docker compose commands

docker compose version

git clone https://github.com/anshulc55/Docker_for_DevOps.git

cd Docker_for_DevOps/docker_compose_build_image/

docker compose build

docker compose up

docker compose up -d (run in background)

docker compose -f Custom-Application.yml up -d (use custom yaml file)

docker compose down

docker compose ls

docker compose push

docker compose logs

docker compose -f Custom-Application.yml logs <service_name>

docker compose logs --tail=10

docker compose exec <service_name> `<shell>`

# Section 10 - Docker Swarm Introduction_Swarm Orchestration

### Introduction

* Master and Worker Nodes
* Based on Raft Algorithm
* Orchestration

### Initialisation and Commands

docker info

docker swarm init --advertise-addr 206.189.179.106

docker swarm --help

docker service --help

### Creating Service on Docker Swarm

docker service create alpine ping www.google.com

docker service ls

docker service ps wtpra6ei8n7v

docker service inspect wtpra6ei8n7v

docker service update wtpra6ei8n7v --replicas 5

docker container ls

docker container rm -f 1b0345c (even if we kill one out of the 5 containers, it will automatically create another one)

docker service rollback wtpra6ei8n7v

### Creating Docker Swarm Cluster

* One main terminal where docker swarm is installed.
* Created another two seperate droplets/terminals(in digitaocean cloud)

docker node --help

docker node ls

docker swarm join-token manager(Do this on the manager machine)

docker swarm join --token SWMTKN-1-14xyysipzdpo6ycp7wff52v0htf2xv3tq8f33dhcs6j1p7x1g1-afxfenr32cf5156xnrc7xbh7x 206.189.179.106:2377(You will get this command. paste this in the worker 1 droplet)

docker swarm join-token worker(and copy paste the token command in docker-02 machine, then it will join as worker)

docker node promote docker-02(doing this from docker-02 itself will give error, coz it cannot promote itself. Doing it from docker-01 will work)

docker node demote docker-01(we can do this from docker-02, coz it was already promoted to manager node)

docker service create --replicas 10 ping www.google.com

docker service create --replicas 10 alpine ping www.google.com

### Visualizing Cluster State using Docker Swarm Visualizer

docker stack deploy -c docker-compose.yml visualizer

docker stack ls

# Section 11 - Docker Swarm Features and Applications

### Networks in Docker Swarm

Default network - Ingress

User can also define networks

Bridge network (docker _gwbridge) - connects individual nodes in the swarm to each other

### Working Example

* Create 3 manager and 2 worker in play with docker site

[manager1]

docker network ls

docker network create -d overlay my_network

docker service create --name postgres -e POSTGRES_PASSWORD=123 --network my_network postgres

docker service create --name drupal --network my_network -p 8080:80 drupal

docker service ps postgress

docker service ps drupal

### Assignment

$ git clone https://github.com/dockersamples/example-voting-app.git~

$ cd example-voting-app/

$ docker network create -d overlay frontend

$ docker network create -d overlay backend

$ docker network ls

$ docker service create --name voting_app -p 5000:80 --replicas 4 --network frontend dockersamples/examplevotingapp_vote

$ docker service create --name redis --replicas 4 --network frontend redis:alpine

$ docker service create --name worker --replicas 4 --network frontend --network backend dockersamples/examplevotingapp_worker

$ docker volume create myvolume

$ docker service create --name postgres -e POSTGRES_PASSWORD=123 --network backend --mount type=volume,source=myvolume,destination=/var/lib/postgresql/data postgres:15-alpine

$ docker service create --name result -p 5001:80 --replicas 1 --network backend dockersamples/examplevotingapp_result

# Section 12 - Docker Swarm Stack Deployment_Multi Service Deployment

### Introduction

docker stack deploy -c docker-compose.yml web_app

docker stack ls

docker stack ps web_app

* **Vertical Scaling** - Increasing instances/replicas (After editing/increasing the number of replicas in the YAML file, execute the same command used to deploy stack. It will be updated automatically)

* **Horizontal Scaling** - increasing resources (When this is done, the current instances will be shutdown, and new ones will be created. But this will happen in a rolling fashion only. All of them together wont be shutdown, so as to keep tha application running.)


docker stack deploy -c docker-stack.yml voting_app

docker stack services voting_app
