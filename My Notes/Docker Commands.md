
# Section 1-4 - Docker Basics and Introduction

###### To check docker setup

```bash
sudo docker run hello-world
```

###### To check all running and stopped containers

```bash
docker ps -a
```

###### To check docker version

```bash
docker --version
docker version
docker info
```

```bash
# General CLI help/examples
docker
docker image --help

# List containers
docker container ls
docker container ls -a

# Search an image on Docker Hub
docker search nginx

# Run NGINX (foreground)
docker container run --publish 8080:80 nginx

# Run NGINX (background)  ← fixed: --detach (not --detatch)
docker container run --publish 8080:80 --detach nginx
docker container run --publish 8081:80 --detach nginx

# Stop / start / remove containers
docker container stop <id>
docker container start <id>
docker container rm <name>
docker container rm -f <name>

# Named container examples
docker container run --publish 8080:80 --detach --name nig1 nginx
docker container run -p 8080:80 -d --name nig1 nginx

# Logs & processes
docker container logs nig1
docker container top nig1
ps -ef | grep nginx

# Quick MySQL test container
docker run --name custom_mysql -e MYSQL_ROOT_PASSWORD=123 -d mysql

# Stats & inspect
docker stats <container_name>
docker inspect <container_name>

# Start an interactive shell in a new NGINX container (publishes 8080)
docker container run -p 8080:80 -it --name nig1 nginx /bin/bash

# Find the default index and view it
find / -name "index.html"
cat /usr/share/nginx/html/index.html
exit

# Exec into running MySQL and login as root
docker exec -it custom_mysql mysql -uroot -p
exit
```

---

# 5. Docker Networking

### Introduction

```bash
docker run -p 8080:80 -d --name nig2 nginx
docker run -p 8081:80 -d --name nig3 nginx

# Get container details (incl. IP)
docker inspect nig2
docker inspect nig3

# Get just the IP address (Go template)
docker inspect -f '{{ .NetworkSettings.IPAddress }}' nig2
docker inspect -f '{{ .NetworkSettings.IPAddress }}' nig3

# Enter nig2
docker exec -it nig2 /bin/bash

# From inside nig2, ping nig3’s IP (default bridge network uses IPs)
ping 172.17.0.3
```

### Docker container Networks

```bash
# Port mapping pattern
docker run -p <host_port>:<container_port> ...

# Show published ports for a container
docker port <container_name>

# Inspect a container’s networking info
docker inspect <container_name>

# Example (IP via format)
docker inspect -f '{{ .NetworkSettings.IPAddress }}' nig2
```

### CLI Operations

```bash
# Create / inspect / remove / prune networks
docker network create <network_name>
docker network inspect <network_name>
docker network rm <network_name>
docker network prune

# Connect / disconnect a container to/from a network (fixed from 'netword stop')
docker network connect <network_name> <container_name>
docker network disconnect <network_name> <container_name>
```

### DNS Concept

```bash
docker network create dns_bridge

docker run -p 8088:80 -d --name alpine_container --network dns_bridge nginx:alpine
docker run -p 8090:80 -d --name alpine_container_2 --network dns_bridge nginx:alpine

# Container-to-container name resolution across same user-defined bridge
docker exec -it alpine_container ping alpine_container_2
```

---

# 6. Docker Container Images: Beginning

### Concept of Docker Image Layers

```bash
docker images
docker pull mysql:8.0
docker history <image_name>   # shows image layers
```

### Concept Image Tagging

```bash
# Tag an image (creates a new tag)
docker tag nginx:latest nginx_noel:1.0.0
```

---

# Section 7. Docker Container Images_Build Container Images

### Docker file format

```dockerfile
FROM <image_name>:<image_tag>
LABEL "<key>"="<value>"
RUN <commands>
CMD ["executable", "param1", "param2"]
EXPOSE <port>
ENV <key>=<value>
ADD <src> <dest>       # add/copy
COPY <src> <dest>      # copy only
VOLUME <paths>         # expose persistent storage areas
WORKDIR <path>         # set working directory
```

### Creating custom docker image

1. Create `dockerfile` in VS Code.
2. Use this Dockerfile (fixed spacing + typo: `upgrade`):
   ```dockerfile
   FROM ubuntu:latest

   LABEL version="1.0"
   LABEL description="This is a sample Dockerfile for building a simple Ubuntu-based image."
   LABEL maintainer="noelmathen03@gmail.com"

   RUN apt-get update && \
       apt-get upgrade -y

   RUN apt-get install -y nginx

   EXPOSE 80

   CMD ["nginx", "-g", "daemon off;"]
   ```
3. Create a working folder:
   ```bash
   mkdir customimages
   ```
4. Create/edit the Dockerfile:
   ```bash
   vi dockerfile   # i -> paste -> Esc -> :wq
   ```
5. Build & list images:
   ```bash
   docker build -t noelmathen/nginx_latest:1.0 .
   docker images
   ```

### Extending docker official image

1. ```bash
   mkdir extendedImage
   vi extendedImage/dockerfile
   cd extendedImage/
   ```
2. Dockerfile:
   ```dockerfile
   FROM nginx:latest

   LABEL version="1.0"
   LABEL description="This is a sample Dockerfile for building a simple Nginx-based image."

   WORKDIR /usr/share/nginx/html
   COPY index.html index.html
   ```
3. Create `index.html`:
   ```html
   <h1>Extended docker image</h1>
   ```
4. Build & run:
   ```bash
   docker build -t noelmathen/nginx_extended:1.0 .
   docker run -it -d -p 8081:80 --name nginx_extended noelmathen/nginx_extended:1.0
   ```

---

# Section 8. Handle Persistent Data in Docker Containers

### Handle with data Volumes

```bash
docker pull mysql:8.2
docker run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:8.2

# (Host path example; your volume ID will differ)
cd /var/lib/docker/volumes/<volume_id>/_data

# Inside the container
docker exec -it mysql /bin/bash
cd /var/lib/mysql
```

> Note: deleting the container does not delete named volume data (persistence).

```bash
docker volume ls
docker volume prune -a
docker volume create <name>
docker volume rm <name>
```

```bash
# Reuse same named volume (explicit mount syntax)
docker run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --mount source=mysql_db,target=/var/lib/mysql \
  --name mysql2 mysql:8.2

# Short -v syntax
docker run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  -v mysql_db:/var/lib/mysql \
  --name mysql3 mysql:8.2
```

### Handle with data binds

Bind mounts can be **any host path** (live sync).

```bash
mkdir ~/testbind

docker run -d \
  --mount type=bind,source=/root/testbind,target=/apps \
  -p 8080:80 --name nginx1 nginx:latest

# New terminal → inspect inside the container
docker exec -it nginx1 /bin/bash
cd /apps && ls
```

> With bind mounts you see real-time changes from host ↔ container.
> With named volumes, multiple containers can share the same data, but you don’t live-edit the volume contents from inside one container when others depend on it.

---

# Section 9 - Docker Compose -Multi Container Orchestration

### Docker-compose YAML file structure example:

```yaml
version: "3.8"

services:
  web:
    build:
      context: ./web
      dockerfile: Dockerfile.dev
      args:
        build_var: "123"
    image: myapp_web:1.0
    container_name: webapp
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=development
      - API_URL=http://api:5000
    env_file:
      - .env
    volumes:
      - ./html:/usr/share/nginx/html
      - app-logs:/var/log/nginx
    depends_on:
      - db
    restart: always
    command: ["nginx", "-g", "daemon off;"]
    entrypoint: ["sh", "-c"]
    working_dir: /app
    extra_hosts:
      - "host.docker.internal:host-gateway"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
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
  db-data:
  app-logs:

networks:
  app-network:
```

### Docker compose commands

```bash
docker compose version

git clone https://github.com/anshulc55/Docker_for_DevOps.git
cd Docker_for_DevOps/docker_compose_build_image/

docker compose build
docker compose up
docker compose up -d                       # run in background
docker compose -f Custom-Application.yml up -d
docker compose down
docker compose ls
docker compose push
docker compose logs
docker compose -f Custom-Application.yml logs <service_name>
docker compose logs --tail=10
docker compose exec <service_name> <shell>
```

---

# Section 10 - Docker Swarm Introduction_Swarm Orchestration

### Introduction

* Master and Worker Nodes
* Based on Raft Algorithm
* Orchestration

### Initialisation and Commands

```bash
docker info
docker swarm init --advertise-addr 206.189.179.106
docker swarm --help
docker service --help
```

### Creating Service on Docker Swarm

```bash
# Example (image + command):
docker service create alpine ping www.google.com

docker service ls
docker service ps wtpra6ei8n7v
docker service inspect wtpra6ei8n7v
docker service update wtpra6ei8n7v --replicas 5

docker container ls
docker container rm -f 1b0345c   # killing one of the tasks; swarm recreates it

docker service rollback wtpra6ei8n7v
```

### Creating Docker Swarm Cluster

* One main terminal where docker swarm is installed.
* Created another two separate droplets/terminals (in DigitalOcean cloud)

```bash
docker node --help
docker node ls

# On the manager:
docker swarm join-token manager
docker swarm join-token worker
```

Run the printed join commands on the other machines, e.g.:

```bash
docker swarm join --token <MANAGER_TOKEN> 206.189.179.106:2377  # join as manager
docker swarm join --token <WORKER_TOKEN>  206.189.179.106:2377  # join as worker
```

Manager role changes:

```bash
docker node promote docker-02
docker node demote docker-01
```

Examples:

```bash
docker service create --replicas 10 alpine ping www.google.com
```

### Visualizing Cluster State using Docker Swarm Visualizer

```bash
# Via stack (compose file must define visualizer)
docker stack deploy -c docker-compose.yml visualizer

docker stack ls
```

---

# Section 11 - Docker Swarm Features and Applications

### Networks in Docker Swarm

- Default network — **ingress**
- Users can also define networks
- Bridge network (`docker_gwbridge`) — connects individual nodes to each other

### Working Example

* Create 3 managers and 2 workers in Play-With-Docker

**[manager1]**

```bash
docker network ls
docker network create -d overlay my_network

docker service create --name postgres -e POSTGRES_PASSWORD=123 --network my_network postgres
docker service create --name drupal --network my_network -p 8080:80 drupal

docker service ps postgres
docker service ps drupal
```

### Assignment

```bash
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app/

docker network create -d overlay frontend
docker network create -d overlay backend
docker network ls

docker service create --name voting_app -p 5000:80 --replicas 4 --network frontend dockersamples/examplevotingapp_vote
docker service create --name redis --replicas 4 --network frontend redis:alpine
docker service create --name worker --replicas 4 --network frontend --network backend dockersamples/examplevotingapp_worker

docker volume create myvolume
docker service create --name postgres -e POSTGRES_PASSWORD=123 --network backend \
  --mount type=volume,source=myvolume,destination=/var/lib/postgresql/data postgres:15-alpine

docker service create --name result -p 5001:80 --replicas 1 --network backend dockersamples/examplevotingapp_result
```

---

# Section 12 - Docker Swarm Stack Deployment_Multi Service Deployment

### Introduction

```bash
docker stack deploy -c docker-compose.yml web_app
docker stack ls
docker stack ps web_app
```

* **Vertical Scaling** – increase replicas (edit YAML then redeploy the same command).
* **Horizontal Scaling** – increase resources (rolling update keeps app available).

```bash
docker stack deploy -c docker-stack.yml voting_app
docker stack services voting_app
docker stack rm mystack
```

---

# Section 13 - Docker Swarm Secrets Management_Protect Sensitive Data

### Secret Management

```bash
mkdir SecretsExample
cd SecretsExample

# Create secret via file
vi pass.txt
docker secret create db_pass pass.txt

# Create secret via stdin
echo "admin" | docker secret create db_admin -

# Inspect
docker secret inspect db_pass
```

```bash
# Example Postgres service consuming secrets (env uses *_FILE)
docker service create --name postgres \
  --secret db_user --secret db_pass \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/db_pass \
  -e POSTGRES_USER_FILE=/run/secrets/db_user \
  postgres
```

```bash
# Files for stack-based secrets
vi postgres_user.txt
vi postgres_password.txt

# Another example via stdin
echo "123" | docker secret create my-secret -

# Deploy stack using secrets from compose
docker stack deploy -c docker-compose.yml postgres_os_db
docker stack ps postgres_os_db
```

---

# Section 14 - Docker Swarm Service Management

### ZeroDowntime Service Upgrade

```bash
docker service create --name web_server -p 8080:80 nginx:1.15.12
docker service scale web_server=10
docker service update --image nginx:latest web_server                # fixed name
docker service update --publish-rm 8080 --publish-add 8090 web_server
```

### HealthCheck in Docker Swarm

* Default health check interval is **30 seconds**.

```bash
docker container run --name postgres1 -d postgres
docker container exec -it postgres1 /bin/bash
pg_isready -U postgres

# Container-level healthcheck (works, but service-level is preferred)
docker container run -d --name postgres2 --health-cmd="pg_isready -U postgres || exit 1" postgres

# Service-level healthcheck (service enters 'starting' until first success)
docker service create --name postgres1 --health-cmd="pg_isready -U postgres || exit 1" postgres
```

### Container Placement in Docker Swarm

* **Service Constraints**

```bash
# Visualizer (single-node quick view)
docker run -it -d -p 8080:80 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer

# Constrain to manager nodes
docker service create --name postgres_db --constraint node.role==manager postgres

# Add a custom node label
docker node update --label-add=region=east-1-d 19z6bef3yd8s4qzmd9xaegk5k

# Place service on nodes matching a label
docker service create --name postgres --constraint node.labels.region==east-1-d postgres

# Update constraints (fixed flags/typos)
docker service update \
  --constraint-rm node.labels.region==east-1-d \
  --constraint-add node.role==worker \
  postgres
```

### Putting Constraints in docker stack

`docker-compose.yml`:

```yaml
version: '3.8'

services:
  mysqlDB:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: "123"
      MYSQL_DATABASE: "mysqldb"
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.region == east-1-d
```

```bash
docker stack deploy -c docker-compose.yml mysql1
```
