# 🐳 Docker Commands Cheat Sheet

---

## ✅ Setup & Verification

```bash
# Check Docker version
docker --version
docker version
docker info

# Check Docker is working
sudo docker run hello-world
```

---

## 📦 Images

```bash
# Search for an image on Docker Hub
docker search nginx

# Get help on image-related commands
docker image --help
```

---

## 🚢 Containers - Basic Usage

```bash
# Run NGINX and map container port 80 to host port 8080
docker container run --publish 8080:80 nginx

# Run NGINX in detached mode (background)
docker container run --publish 8080:80 --detach nginx

# Run a second NGINX container on a different host port
docker container run --publish 8081:80 --detach nginx

# Run NGINX with custom container name
docker container run --publish 8080:80 --detach --name nig1 nginx

# Same as above, using shorthand flags
docker container run -p 8080:80 -d --name nig1 nginx
```

---

## 📋 Container Management

```bash
# List running containers
docker container ls

# List all containers (including stopped)
docker container ls -a
docker ps -a

# Stop a running container
docker container stop <container_id_or_name>

# Start a stopped container
docker container start <container_id_or_name>

# Remove a container by name
docker container rm <container_name>

# Force remove a container
docker container rm -f <container_name>
```

---

## 🧠 Container Monitoring & Logs

```bash
# View logs of a container
docker container logs nig1

# Show running processes inside a container
docker container top nig1

# System-wide process check for nginx
ps -ef | grep nginx

# Live CPU/RAM usage of containers
docker stats <container_name>
```

---

## 🧪 Inspect & Debug

```bash
# Inspect detailed metadata of a container
docker inspect <container_name>
```

---

## 🐬 MySQL Container Example

```bash
# Run MySQL container with custom name and root password
docker run --name custom_mysql -e MYSQL_ROOT_PASSWORD=123 -d mysql

# Access MySQL inside the container
docker exec -it custom_mysql mysql -uroot -p
# (then type the password: 123)
exit
```

---

## 🧭 Interactive Bash Access (Debugging)

```bash
# Run a container interactively with bash (requires image to support it)
docker container run -p 8080:80 -it nig1 nginx /bin/bash

# Search for index.html file inside container
find / -name "index.html"

# View content of the default nginx welcome page
cat /usr/share/nginx/html/index.html

# Exit container bash shell
exit
```
