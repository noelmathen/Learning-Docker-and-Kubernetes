
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
