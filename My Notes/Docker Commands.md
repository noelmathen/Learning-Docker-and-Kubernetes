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
