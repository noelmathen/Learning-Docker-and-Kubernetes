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

# Section 15 - Get Started with Kubernetes

### Architecture Overview

* Master - Slave
* Kube-api-server (Handles requests as REST by master?)
* Etcd - Backend of K8s
* Kube Scheduler
* Kube Control Manager(Manages automated processes)
* Kubelet - K8s agent executed on worker nodes.
* Pods - Contains 1 or more containers. Share same network, storage etc. But All the containers in the pod should be same configuration. cant be different.
* Kube Proxy - Runs on each node to deal with individual host sub-netting and ensure that the services are available to external parties.

### Installation

wsl --version

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes.gpg
echo "deb https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl

kubectl version --client

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube start --driver=docker

kubectl get nodes

kubectl config view

### Some Basic Commands

kubectl create deployment hello-node --image=k8s.gcr.ic/echoserver:1.4

kubectl get deployment

kubectl get pods

kubectl expose deployment hello-node --type=LoadBalancer --port=8080

minikube service hello-node

kubectl delete service hello-node

kubectl delete deployment hello-node

### Namespaces

kubectl get pods --namespace kube-system

kubectl get pods --all-namespaces

kubectl create namespace levelup360

kubectl get namespaces

# Section 16 - Kubernetes Cluster Management

### High Availability

* Availablity of k8s cluster
* Uses load balancer, multiple masters and workers
* Stacked ETCD - each master nodes have their own etcd, and these etcds communicate with eo
* External Etcd - the etcs are maintained outside, not inside the masters

### K8s Management Tools

* Kubectl
  * Official CLI
  * Or we can use REST API
* Kubeadm
  * Use to create k8s cluster
* Minikube
  * Help setup master and worker node in single machine
* Helm
  * K8s templte andpackage management
  * Use k8s as reusable objects
* Kompose
  * Convert docker copmose files to k8sobjects
  * Ship containers from compose to k8s
* Kustomize
  * Similar to helm

### K8s HA Cluster Setup

### Step 1: Prerequisites on ALL Three Nodes (`master`, `worker-1`, `worker-2`)

You must run these exact commands on **all three** of your virtual machines. This ensures they all have the necessary container runtime and Kubernetes packages.

#### 1. Prepare the System

First, we'll update the package list and set up the necessary kernel modules and system settings for Kubernetes networking.

**Bash**

```
# Update package lists
sudo apt-get update

# Configure required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params for networking
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

#### 2. Install and Configure `containerd` Runtime

Kubernetes needs a container runtime to manage containers. We'll use `containerd`.

**Bash**

```
# Install containerd
sudo apt-get install -y containerd

# Create a default configuration file
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# IMPORTANT: Set the SystemdCgroup to true
# This is required for kubelet to work correctly
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Restart containerd to apply the new config
sudo systemctl restart containerd

# Verify that containerd is running
sudo systemctl status containerd
```

*You should see an **active (running)** status.*

#### 3. Disable Swap

Kubernetes requires that you disable swap memory on all nodes.

**Bash**

```
# Disable swap immediately
sudo swapoff -a

# Disable swap permanently in the fstab file
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

#### 4. Install Kubernetes Packages (`kubelet`, `kubeadm`, `kubectl`)

Now, we'll add the official Kubernetes package repository and install the tools.

**Bash**

```
# Install dependencies
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Add Kubernetes official GPG key (using the new method)
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package list and install Kubernetes tools
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Mark the packages to prevent accidental updates
sudo apt-mark hold kubelet kubeadm kubectl
```

**Checkpoint:** At this point, all three of your nodes are ready. The next steps are specific to the master or worker nodes. ✅

---

### Step 2: Initialize the Control-Plane Node (Run ONLY on `master`)

These commands initialize the cluster and should  **only be run on your `master` node** .

#### 1. Initialize the Cluster

This command bootstraps your Kubernetes cluster, sets up the control plane components, and specifies the IP range for your pods.

**Bash**

```
# Use kubeadm to initialize the cluster
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

After this command finishes, it will print a `kubeadm join` command at the end of its output. **Copy this entire command and save it somewhere safe.** You will need it to join your worker nodes to the cluster. It will look something like this:

`kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

#### 2. Configure `kubectl` Access

To manage your cluster as a regular user, you need to set up the kubeconfig file.

**Bash**

```
# Create the .kube directory in your home folder
mkdir -p $HOME/.kube

# Copy the admin config file to your .kube directory
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# Change the ownership of the file to your user
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

You can now test access by running `kubectl get nodes`. The master node will show a `NotReady` status because a networking add-on has not been installed yet. This is expected.

---

### Step 3: Install a Pod Network Add-on (Run ONLY on `master`)

Your pods need a way to communicate with each other across nodes. We'll install the Calico network add-on.

**Bash**

```
# Apply the Calico CNI manifest
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

Wait for 2-3 minutes for all the Calico pods to start. You can check the status with:

**Bash**

```
# Check the status of your nodes again
kubectl get nodes
```

The master node's status should soon change from `NotReady` to  **`Ready`** . 👨‍💻

---

### Step 4: Join the Worker Nodes (Run on `worker-1` and `worker-2`)

Now, it's time to add your worker nodes to the cluster.

1. **Get the Join Command:** Find the `kubeadm join` command you saved earlier from the `kubeadm init` output. If you lost it, you can generate a new one by running this on the  **master node** :
   **Bash**

   ```
   kubeadm token create --print-join-command
   ```
2. **Join the Nodes:** SSH into `worker-1`, paste the full `kubeadm join` command, and run it with `sudo`.
   **Bash**

   ```
   # On worker-1
   sudo kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```
3. Repeat the same step on `worker-2`.
   **Bash**

   ```
   # On worker-2
   sudo kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

---

### Step 5: Final Verification (Run on `master`)

Return to your **master node** and check the status of all nodes in the cluster.

**Bash**

```
# Get the status of all nodes with more details
kubectl get nodes -o wide
```

You should now see all three nodes (`master`, `worker-1`, `worker-2`) listed with a **`Ready`** status.

```
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master     Ready    control-plane   10m   v1.30.x   10.0.0.1       <none>        Ubuntu 22.04.3 LTS   5.15.0-78-generic   containerd://1.6.21
worker-1   Ready    <none>          2m    v1.30.x   10.0.0.2       <none>        Ubuntu 22.04.3 LTS   5.15.0-78-generic   containerd://1.6.21
worker-2   Ready    <none>          1m    v1.30.x   10.0.0.3       <none>        Ubuntu 22.04.3 LTS   5.15.0-78-generic   containerd://1.6.21
```

Congratulations! You have successfully set up a modern, multi-node Kubernetes cluster. ✨

### Maintenance

* Node Draining - Here, basically an existing node can be drained. any pods running in it will be gracefully terminated, and will be reassigned to another node.
* We can undrain it as well. When drained, it will not be scheduled for anmmy tasks

mkdir node_draining

cd node_draining

vi pods.yaml

apiVersion: v1
kind: Pod
metadata:
  name: draining-node-test-pod
  labels:
    tier: frontend
spec:
  restartPolicy: OnFailure
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80

kubectly apply -f pods.yaml

kubectl get pods -o wide

vi deployment.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: draining-node-test-deployment

  labels:

    app: frontend

spec:

  replicas: 2

  selector:

    matchLabels:

    app: frontend

  template:

    metadata:

    labels:

    app: frontend

    spec:

    containers:

    - name: nginx

    image: nginx:latest

    ports:

    - containerPort: 8080

kubectly apply -f deplyment.yaml

kubectl drain worker-2 --ignore-daemonsets --force

kubectl get nodes

kubectl get pods -o wide

kubectl uncordon worker-2

### Upgrading The K8s Cluster

Of course. Upgrading a Kubernetes cluster is a critical operation, and doing it correctly is essential. The process you've outlined is generally correct in its flow, but we'll modernize it for your current **v1.30** cluster and incorporate best practices.

This guide will walk you through upgrading your cluster from **v1.30.x** to the next minor version, **v1.31.y** (this is a hypothetical future version for demonstration, but the process is identical for any minor version upgrade).

**Important Rule:** You can only upgrade one minor version at a time (e.g., 1.30 → 1.31). You cannot skip versions (e.g., 1.30 → 1.32).

---

### Part 1: Upgrade the Control-Plane Node (`master`)

The control-plane **must** be upgraded first. This process will update `etcd`, the API server, and other critical components.

#### 1. Check Current Status & Plan the Upgrade

First, see the current version of all nodes to confirm your starting point.

**Bash**

```
# Run this on the master node
kubectl get nodes
```

You should see all nodes are on v1.30.x. Before any upgrade, it's highly recommended to **read the official release notes** for the target version (v1.31) to be aware of any deprecated APIs or breaking changes.

#### 2. Update the Kubernetes Package Repository

To get the new version's packages, you must update the repository source file on the node you're upgrading.

**Bash**

```
# Run this on the master node
# This command updates your repository from v1.30 to v1.31
sudo sed -i 's/v1.30/v1.31/g' /etc/apt/sources.list.d/kubernetes.list

# Update your local package index
sudo apt-get update
```

#### 3. Upgrade `kubeadm`

`kubeadm` is the tool that orchestrates the upgrade. It needs to be upgraded first.

**Bash**

```
# Define the target version
# NOTE: Use the latest patch release of v1.31 available. Check with `apt-cache madison kubeadm`.
VERSION=1.31.1-00

# Install the new version of kubeadm
sudo apt-get install -y --allow-change-held-packages kubeadm=$VERSION

# Verify the new version
kubeadm version
```

#### 4. Drain the Master Node

Draining the node safely evicts all your application pods, moving them to other available nodes. This ensures zero downtime for your apps.

**Bash**

```
# Replace <master-node-name> with the actual name of your master node
kubectl drain <master-node-name> --ignore-daemonsets
```

* **`--ignore-daemonsets`** is needed because DaemonSet pods run on every node and cannot be evicted.

#### 5. Perform the Control-Plane Upgrade

Now, use `kubeadm` to perform the actual upgrade.

**Bash**

```
# First, see the plan. This is a safe check.
sudo kubeadm upgrade plan

# Then, apply the upgrade. This will back up etcd and upgrade the static pods.
sudo kubeadm upgrade apply v1.31.1
```

Follow any instructions the command outputs. It may ask you to upgrade other components if needed.

#### 6. Upgrade `kubelet` and `kubectl`

Once the control-plane components are upgraded, you can upgrade the `kubelet` (the node agent) and `kubectl` (the CLI tool).

**Bash**

```
# Install the new versions
sudo apt-get install -y --allow-change-held-packages kubelet=$VERSION kubectl=$VERSION

# Reload the systemd config and restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

#### 7. Uncordon the Master Node

"Uncordoning" makes the node available again for scheduling new pods.

**Bash**

```
# Make the node schedulable again
kubectl uncordon <master-node-name>
```

#### 8. Verify the Upgrade ✅

Check the nodes again. Your master node should now report the new version (v1.31.1) and be in a `Ready` state.

**Bash**

```
kubectl get nodes
```

---

### Part 2: Upgrade the Worker Nodes (`worker-1` & `worker-2`)

Now, upgrade each worker node, one at a time. The process is similar but slightly different.

#### **On `worker-1`:**

1. **Update the Package Repository (on the worker):** SSH into `worker-1` and update its repository to point to v1.31.
   **Bash**

   ```
   # Run this on worker-1
   sudo sed -i 's/v1.30/v1.31/g' /etc/apt/sources.list.d/kubernetes.list
   sudo apt-get update
   ```
2. **Drain the Worker Node (from the master):** Go back to your **master node's** terminal to drain the worker.
   **Bash**

   ```
   # Run this on the master node
   kubectl drain <worker-1-name> --ignore-daemonsets
   ```
3. **Upgrade `kubeadm` (on the worker):** Go back to the `worker-1` terminal.
   **Bash**

   ```
   # Run this on worker-1
   VERSION=1.31.1-00
   sudo apt-get install -y --allow-change-held-packages kubeadm=$VERSION
   ```
4. **Upgrade the Kubelet Config (on the worker):** For worker nodes, you run `kubeadm upgrade node`.
   **Bash**

   ```
   # Run this on worker-1
   sudo kubeadm upgrade node
   ```
5. **Upgrade `kubelet` (on the worker):**
   **Bash**

   ```
   # Run this on worker-1
   sudo apt-get install -y --allow-change-held-packages kubelet=$VERSION
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```
6. **Uncordon the Worker Node (from the master):** Return to the **master** terminal.
   **Bash**

   ```
   # Run this on the master node
   kubectl uncordon <worker-1-name>
   ```

#### **Repeat for `worker-2`:**

Follow the exact same steps (1-6) for your second worker node, `worker-2`, replacing the node name where appropriate.

---

### Final Verification 🚀

Once all nodes are upgraded, run this command on your master node one last time:

**Bash**

```
kubectl get nodes -o wide
```

All three of your nodes should now be **`Ready`** and report being on the new version,  **v1.31.1** . Your cluster upgrade is complete!

# Section 17 - Kubernetes Object Management

### Working with Kubectl

* Command line tool
* Use k8s API internally
* Kubectl get - Get objects in K8s cluster
* Kubectl describe - Detailed information about object
* Kubectl create - Create object
* Kubectl apply - Similar to create - MOdify existing object is possible
* Kubectl delete
* Kubectl exec - Used to execute commands inside container. Resource shohuld be in running state

mkdir object_management

cd object_management/

vi pod.yml

kubectl create -f pod.yml

kubectl get

kubectl api-resources

kubectl get po

keubectl get pods -n kube-system

kubectl get pods -n kube-system

kubectl get pods

kubectl get pods draining-node-test-pod

kubectl get pods draining-node-test-pod -o wide

kubectl get pods draining-node-test-pod -o json

kubectl describe pods draining-node-test-pod

kubectl exec draining-node-test-pod -c nginx -- cat /etc/nginx/nginx.conf

kubectl delete po draining-node-test-pod

kubectl get pods

### Role Based Access Management in K8s

* Admin can control the user access
* "Roles" and "ClusterRoles" are k8s object which manages permissions
* Roles - withing namespace
* ClusterRoles - Across the entire cluster
* "RoleBinding" - connects roles to user
* "ClusterBinding" - ClusterRole mapping to user

### Service Accounts in K8s Cluster

* Used by container porcess to authenticate with K8s API
* Can create service account using YAML file
* Define role binding

kubectl get serviceaccounts

kubectl get serviceaccounts -n development

vi my-serviceaccount.yml

kubectl apply -f my-serviceaccount.yml

kubectl get serviceaccounts -n development

kubectl get roles -n development

vi service-account-binding.yml

kubectl apply -f service-account-binding.yml

kubectl get serviceaccounts -n development

kubectl get rolebinding -n development

# Section 18 - Pods and Containers in Kubernetes

### Application Configuration

* Kubernetes allows user to pass dynamic configuration values to application at Runtime.
* ConfigMap

  * Contains non sensitive data
  * Passes to container application
  * key-value format
  * seperate configurations from pds and copmonents
  * Flexible, prevents hardcoding into pods.
* sECRETS

  * Stores sensitive data
  * For the characters, $, \, *, !, we need escaping
* Environement Variables - Used to pass configmap and secrets to containers
* Mount Volume - Same thing

### Manage Applications using ENV Variables

mkdir pods_and_containers && cd pods_and_containers

vi example-configMap.yml

apiVersion: v1

kind: ConfigMap

```
metadata:

  name: player-pro-demo

data:

  player_lives: "5"

  properties_file_name: "user-interface.properties"

  base.properties: |

    enemy.types=aliens,monsters

    player.maximum-lives=10

  user-interface.properties: |

    color.good=purple

    color.bad=yellow

    allow.textmode=true
```

kubectl apply -f example-configMap.yml

kubectl get configmaps

kubectl describe configmap player-pro-demo

echo -n 'admin' | base64

vi example-secrect.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-demo
spec:
  containers:
    - name: configmap-demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_LIVES
          valueFrom:
            configMapKeyRef:
              name: player-pro-demo  # The ConfigMap this value comes from.
              key: player_lives # The key to fetch.
        - name: PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: player-pro-demo
              key: properties_file_name
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: example-secret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: example-secret
              key: password
```

kubectl apply -f example-secrect.yml

kubectl get secret

kubectl describe secret example-secret

vi configmap-env-demo.yml

kubectl apply -f configmap-env-demo.yml

kubectl get pods

kubectl exec configmap-env-demo -it -- sh

### Manage Applications using Mount Volumes

vi configmap-vol-demo.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-vol-demo
spec:
  containers:
    - name: configmap-vol-demo
      image: alpine
      command: ["sleep", "3600"]
      volumeMounts:
      - name: player-map
        mountPath: /etc/config/configMap
      - name: player-secret
        mountPath: /etc/config/secret
  volumes:
    # You set volumes at the Pod level, then mount them into containers inside that Pod
    - name: player-map
      configMap:
        # Provide the name of the ConfigMap you want to mount.
        name: player-pro-demo
    - name: player-secret
      secret:
        secretName: example-secret


```

kubectl apply -f configmap-vol-demo.yml

kubectl exec configmap-vol-demo -it -- sh

* Inside the container, You can view the location and file contents where the configmaps and secrets are mounted using the YAML file.

### Manage Application Configuration Posix ConfigMap

vi example-posix-configMap.yml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: player-posix-demo
data:
  PLAYER_LIVES: "5"
  PROPERTIES_FILE_NAME: "user-interface.properties"
  BASE_PROPERTIES: "Template1"
  USER_INTERFACE_PROPERTIES: "Dark"
```

kubectl apply -f example-posix-configMap.yml

kubectl get configmaps

kubectl describe configmap player-pro-demo

vi configmap-posix-demo.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-posix-demo
spec:
  containers:
    - name: configmap-posix
      image: anshuldevops/kubernetes-web:1.10.6
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: player-posix-demo
```

kubectl apply -f configmap-posix-demo.yml

kubectl describe pod configmap-posix-demo

kubectl get pods

kubectl exec configmap-posix-demo -t -- /bin/bash

### ConfigMap and Secret from File

sudo apt-get update

apt install apache2-utils

htpasswd -c .htpasswd user => INPUT A PASSWORD, AND REMEMEBR IT

cat .htpasswd

kubectl create secret generic nginx-htpasswd --from-file .htpasswd

kubectl get secret

kubectl describe secret nginx-htpasswd

rm -rf .htpasswd

vi nginx.conf

```
user nginx
worker_processes auto

error_log /var/log/nginx/error.log notice
pid /var/run/nginx.pid

events
{
  worker_connections 1024
}

http
{
  server
  {
    listen 80
    server_name localhost

    location /
    {
      root /usr/share/nginx/html
      index index.html index.htm
    }

    auth_basic "Secure Site"
    auth_basic_user_file conf/.htpasswd
  }
}

```

kubectl create configmap nginx-config-file --from-file nginx.conf

kubectl describe configmap nginx-config-file

vi nginx-pod.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: 'nginx:1.19.1'
      ports:
        - containerPort: 80
      volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx
        - name: htpasswd-volume
          mountPath: /etc/nginx/conf
  volumes:
    - name: nginx-config-volume
      configMap:
        name: nginx-config-file
    - name: htpasswd-volume
      secret:
        secretName: nginx-htpasswd
```

kubectl apply -f nginx-pod.yml

kubectl get pods

curl -u user:123 192.168.226.71


### Manage Container Resources in K8s

* Resource Request - Preferred usage. May use above or below this
* ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend-1
  spec:
    containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend-2
  spec:
    containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend-3
  spec:
    containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend-4
  spec:
    containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
  ```
* Resource Limit - Actual limit. Wont use above this.

  * If CPU limit exceed - Container still run
  * If memory exceed - Container killed, and restarted according to restart policy
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend-limit
    spec:
      containers:
      - name: app
        image: alpine
        command: ["sleep", "3600"]
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
    ```


### **Monitor Container Resources in K8s**

* Container Heath
* Liveness Probe(active helathcheck)

  * Determine Container State
  * Run commands in container
  * Or periodic HTTP healthcheck
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: liveness-probe
    spec:
      containers:
        - name: liveness
          image: k8s.gcr.io/busybox
          args:
            - /bin/sh
            - -c
            - touch /tmp/healthcheck; sleep 60; rm -rf /tmp/healthcheck; sleep 600
          livenessProbe:
            exec:
              command:
                - cat
                - /tmp/healthcheck
            initialDelaySeconds: 5
            periodSeconds: 5

    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: liveness-probe-http
    spec:
      containers:
        - name: liveness-nginx
          image: k8s.gcr.io/nginx
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 3
    ```
* Startup Probe

  * Runs at container startup, and stops when container stops running
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: startup-probe-http
    spec:
      containers:
        - name: startup-nginx
          image: k8s.gcr.io/nginx
          startupProbe:
            httpGet:
              path: /
              port: 80
            failureThreshold: 30
            periodSeconds: 10
    ```
* Readiness probe

  * Used to detect if containers are ready to accept traffic
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: hc-probe
    spec:
      containers:
        - name: nginx
          image: k8s.gcr.io/nginx
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 3
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 3
    ```


### Pod Restart Policies

* K8s restart containers if they fail
* We can customize it according to our needs
* THree restart policies: Always, OnFailure, Never
* Always
  * Default policies
  * Even if container completed successfully, it will restart
  * Recommended for always ion running state
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: restart-always-pod
    spec:
      restartPolicy: Always
      containers:
        - name: app
          image: alpine
          command: ["sleep", "10"]
    ```
* OnFailure
  * Works if container exits wiht errorcode
  * Works wiht liveness probe
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: onfailure-always-pod
    spec:
      restartPolicy: OnFailure
      containers:
        - name: app
          image: alpine
          command: ["sleep", "10"]
    ```
* Never
  * It will never be restarted
  * For applications that need to be run only once
  * ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: never-always-pod
    spec:
      restartPolicy: Never
      containers:
        - name: app
          image: alpine
          command: ["sleep", "10"]
    ```



### MultiContainer Pods

* Contianers can share the resources of the pod
* Best practice is to keep single contianer in single pod
* Can use same shared network, shared volume etc etc.
* Connect and communicate through localhost
* ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: two-containers
  spec:
    restartPolicy: OnFailure
    containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
          - name: shared-data
            mountPath: /usr/share/nginx/html
      - name: debian-container
        image: debian
        volumeMounts:
          - name: shared-data
            mountPath: /pod-data
        command: ["/bin/sh"]
        args: ["-c", "echo Hello from the Secondary container > /pod-data/index.html"]
    volumes:
      - name: shared-data
        emptyDir: {}
  ```



### Container Initialisation

* Runs before app containers
* Execute only once
* Specialised instructions/utilities/setup scripts
* we can define N number of init containers
* Init containers offer a mechanism to block or delay app container startup until a set of preconditions are met.
* ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: application-pod
  spec:
    containers:
      - name: myapp-container
        image: busybox:1.28
        command: ["sh", "-c", "echo The app is running! && sleep 3600"]
    initContainers:
      - name: init-myservice
        image: busybox:1.28
        command:
          [
            "sh",
            "-c",
            "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 5; done",
          ]
      - name: init-mydb
        image: busybox:1.28
        command:
          [
            "sh",
            "-c",
            "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 5; done",
          ]
  ```
