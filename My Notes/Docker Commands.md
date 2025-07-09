
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



#
