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

* **Control plane and workers**

  A Kubernetes cluster has a **control plane** (brains) and **worker nodes** (muscle).
* **kube-apiserver**

  Front door of the cluster. Exposes a REST API and validates/serves requests.
* **etcd**

  Highly available key-value store that holds the cluster’s state.
* **kube-scheduler**

  Decides which node each Pod should run on based on resources, constraints, and policies.
* **kube-controller-manager**

  Runs controllers that reconcile actual state to desired state (Deployments, Nodes, endpoints, etc.).
* **kubelet**

  Agent on each worker node. Talks to the API server and ensures containers are running as specified.
* **Pods**

  Smallest deployable unit. A Pod can run **one or more containers** that share network and storage. Containers in a Pod can be different; they are co-located and tightly coupled.
* **kube-proxy**

  Node-level networking component that programs iptables/ipvs to implement Kubernetes Services and in-cluster load balancing.

### Installation

```bash
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
```

### Some Basic Commands

```bash
kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4

kubectl get deployment

kubectl get pods

kubectl expose deployment hello-node --type=LoadBalancer --port=8080

minikube service hello-node

kubectl delete service hello-node

kubectl delete deployment hello-node
```

### Namespaces

```bash
kubectl get pods --namespace kube-system

kubectl get pods --all-namespaces

kubectl create namespace levelup360

kubectl get namespaces
```

---

# Section 16 - Kubernetes Cluster Management

### High Availability

* **Availability of the Kubernetes cluster** using multiple control planes and workers behind a load balancer.
* **Stacked etcd** : each control-plane node runs its own etcd; etcd members form a cluster.
* **External etcd** : etcd runs outside the control plane nodes and is managed separately.

### K8s Management Tools

* **kubectl** : official CLI. You can also talk directly to the REST API.
* **kubeadm** : bootstraps Kubernetes clusters.
* **Minikube** : runs a local single-machine cluster for learning and dev.
* **Helm** : package manager and templating for Kubernetes.
* **Kompose** : converts Docker Compose to Kubernetes manifests.
* **Kustomize** : configuration overlays; template-free customization.

### K8s HA Cluster Setup

#### Step 1: Prerequisites on ALL Three Nodes (`master`, `worker-1`, `worker-2`)

**1) Prepare the system**

```bash
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

**2) Install and configure `containerd`**

```bash
# Install containerd
sudo apt-get install -y containerd

# Create a default configuration file
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# IMPORTANT: Set SystemdCgroup = true
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd

# Verify
sudo systemctl status containerd
```

**3) Disable swap**

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

**4) Install Kubernetes packages (`kubelet`, `kubeadm`, `kubectl`)**

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Step 2: Initialize the Control-Plane Node (Run ONLY on `master`)

**1) Initialize the cluster**

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

Copy the `kubeadm join ...` command printed at the end for later.

**2) Configure `kubectl` for your user**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Step 3: Install a Pod Network Add-on (Run ONLY on `master`)

**Calico CNI**

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml

kubectl get nodes
```

#### Step 4: Join the Worker Nodes (Run on `worker-1` and `worker-2`)

**Get a fresh join command if needed (on master):**

```bash
kubeadm token create --print-join-command
```

**Join each worker:**

```bash
# On worker-1
sudo kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

# On worker-2
sudo kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

#### Step 5: Final Verification (Run on `master`)

```bash
kubectl get nodes -o wide
```

### Maintenance

* **Node draining**

  Draining a node gracefully evicts schedulable Pods so workloads reschedule elsewhere. You can later mark the node schedulable again.

**Example manifests and workflow**

```yaml
# pods.yaml
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
```

```bash
mkdir node_draining
cd node_draining

kubectl apply -f pods.yaml
kubectl get pods -o wide
```

```yaml
# deployment.yaml
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
```

```bash
kubectl apply -f deployment.yaml

kubectl drain worker-2 --ignore-daemonsets --force
kubectl get nodes
kubectl get pods -o wide
kubectl uncordon worker-2
```

### Upgrading The K8s Cluster

> Upgrade one minor version at a time (e.g.,  **1.30 → 1.31** ).

#### Part 1: Upgrade the Control-Plane Node (`master`)

**1) Check status**

```bash
kubectl get nodes
```

**2) Update repo on master to target version (example: v1.31)**

```bash
sudo sed -i 's/v1.30/v1.31/g' /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

**3) Upgrade `kubeadm` first**

```bash
VERSION=1.31.1-00
sudo apt-get install -y --allow-change-held-packages kubeadm=$VERSION
kubeadm version
```

**4) Drain the master**

```bash
kubectl drain <master-node-name> --ignore-daemonsets
```

**5) Plan and apply the control-plane upgrade**

```bash
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.31.1
```

**6) Upgrade `kubelet` and `kubectl` on master**

```bash
sudo apt-get install -y --allow-change-held-packages kubelet=$VERSION kubectl=$VERSION
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

**7) Uncordon the master**

```bash
kubectl uncordon <master-node-name>
```

**8) Verify**

```bash
kubectl get nodes
```

#### Part 2: Upgrade the Worker Nodes (`worker-1` & `worker-2`), one at a time

**On `worker-1`: update repo and packages**

```bash
# On worker-1
sudo sed -i 's/v1.30/v1.31/g' /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

**Drain from master**

```bash
# On master
kubectl drain <worker-1-name> --ignore-daemonsets
```

**Upgrade on worker-1**

```bash
# On worker-1
VERSION=1.31.1-00
sudo apt-get install -y --allow-change-held-packages kubeadm=$VERSION
sudo kubeadm upgrade node
sudo apt-get install -y --allow-change-held-packages kubelet=$VERSION
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

**Uncordon from master**

```bash
# On master
kubectl uncordon <worker-1-name>
```

**Repeat the same steps for `worker-2`.**

#### Final Verification

```bash
kubectl get nodes -o wide
```

All nodes should be `Ready` and on the target version.

# Section 17 - Kubernetes Object Management

### Working with kubectl

* **kubectl** is the CLI that talks to the Kubernetes API.
* Common verbs:
  * `kubectl get` — list objects
  * `kubectl describe` — detailed view of an object
  * `kubectl create` — create from manifest
  * `kubectl apply` — create/update declaratively (preferred)
  * `kubectl delete` — delete objects
  * `kubectl exec` — run a command inside a container (target Pod must be Running)

```bash
mkdir object_management
cd object_management/

vi pod.yml

kubectl create -f pod.yml

kubectl get
kubectl api-resources

kubectl get po
kubectl get pods -n kube-system
kubectl get pods

kubectl get pods draining-node-test-pod
kubectl get pods draining-node-test-pod -o wide
kubectl get pods draining-node-test-pod -o json

kubectl describe pods draining-node-test-pod

kubectl exec draining-node-test-pod -c nginx -- cat /etc/nginx/nginx.conf

kubectl delete po draining-node-test-pod
kubectl get pods
```

### Role-Based Access Management in K8s

* Use **Roles** and **ClusterRoles** to define permissions.
  * **Role** : namespace-scoped permissions
  * **ClusterRole** : cluster-wide permissions
* Bind permissions to users/service accounts via:
  * **RoleBinding** (namespace)
  * **ClusterRoleBinding** (cluster)

### Service Accounts in K8s Cluster

* Used by in-cluster workloads to authenticate to the API server.
* Typical flow: create a  **ServiceAccount** , define a  **Role** , and bind them with a  **RoleBinding** .

```bash
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
```

---

# Section 18 - Pods and Containers in Kubernetes

### Application Configuration

* Kubernetes lets you inject configuration at runtime.
* **ConfigMap**
  * Non-sensitive data
  * Key/value entries or file-like blobs
  * Decouples config from images and Pods
* **Secret**
  * Sensitive data (base64-encoded; treat as confidential)
  * Be careful with shell-special characters like `$ \ * !` (quote/escape as needed)
* **Environment variables** and **volume mounts** are the two main delivery mechanisms to containers.

### Manage Applications using Environment Variables

```bash
mkdir pods_and_containers && cd pods_and_containers

vi example-configMap.yml
```

```yaml
apiVersion: v1
kind: ConfigMap
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

```bash
kubectl apply -f example-configMap.yml
kubectl get configmaps
kubectl describe configmap player-pro-demo

echo -n 'admin' | base64   # example: how a secret value is encoded

vi example-secrect.yml
```

```yaml
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
        - name: PLAYER_LIVES
          valueFrom:
            configMapKeyRef:
              name: player-pro-demo
              key: player_lives
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

```bash
kubectl apply -f example-secrect.yml
kubectl get secret
kubectl describe secret example-secret

vi configmap-env-demo.yml
kubectl apply -f configmap-env-demo.yml
kubectl get pods
kubectl exec configmap-env-demo -it -- sh
```

> Note: Ensure `example-secret` exists (e.g., via `kubectl create secret generic example-secret --from-literal=username=... --from-literal=password=...`) before creating the Pod that references it.

### Manage Applications using Mount Volumes

```bash
vi configmap-vol-demo.yml
```

```yaml
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
    - name: player-map
      configMap:
        name: player-pro-demo
    - name: player-secret
      secret:
        secretName: example-secret
```

```bash
kubectl apply -f configmap-vol-demo.yml
kubectl exec configmap-vol-demo -it -- sh
```

* Inside the container, inspect `/etc/config/configMap` and `/etc/config/secret` to see files created from ConfigMap/Secret keys.

### Manage Application Configuration: POSIX-Style ConfigMap

```bash
vi example-posix-configMap.yml
```

```yaml
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

```bash
kubectl apply -f example-posix-configMap.yml
kubectl get configmaps
kubectl describe configmap player-pro-demo

vi configmap-posix-demo.yml
```

```yaml
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

```bash
kubectl apply -f configmap-posix-demo.yml
kubectl describe pod configmap-posix-demo
kubectl get pods
kubectl exec configmap-posix-demo -t -- /bin/bash
```

### ConfigMap and Secret from File

```bash
sudo apt-get update
apt install apache2-utils

# Create a basic-auth file. You will be prompted for a password; remember it.
htpasswd -c .htpasswd user

cat .htpasswd

kubectl create secret generic nginx-htpasswd --from-file .htpasswd
kubectl get secret
kubectl describe secret nginx-htpasswd

rm -rf .htpasswd

vi nginx.conf
```

```nginx
user nginx
worker_processes auto

error_log /var/log/nginx/error.log notice
pid /var/run/nginx.pid

events {
  worker_connections 1024
}

http {
  server {
    listen 80
    server_name localhost

    location / {
      root /usr/share/nginx/html
      index index.html index.htm
    }

    auth_basic "Secure Site"
    auth_basic_user_file conf/.htpasswd
  }
}
```

```bash
kubectl create configmap nginx-config-file --from-file nginx.conf
kubectl describe configmap nginx-config-file

vi nginx-pod.yml
```

```yaml
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

```bash
kubectl apply -f nginx-pod.yml
kubectl get pods

# Example access (adjust IP/port for your environment)
curl -u user:123 192.168.226.71
```

### Manage Container Resources in K8s

* **Resource requests** : scheduler’s target/guarantee
* **Resource limits** : hard caps
* If a container exceeds  **CPU limit** : throttled, keeps running
* If a container exceeds  **memory limit** : OOMKilled; restarts based on restart policy

```yaml
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

```yaml
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

### Monitor Container Resources in K8s

* **Container health** can be tracked with probes:
  * **Liveness probe** : determines if the container should be restarted
  * **Startup probe** : gives apps time to initialize before liveness starts
  * **Readiness probe** : signals when a Pod is ready to receive traffic

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-probe
spec:
  containers:
    - name: liveness
      image: k8s.gcr.io/busybox
      args: ["/bin/sh", "-c", "touch /tmp/healthcheck; sleep 60; rm -rf /tmp/healthcheck; sleep 600"]
      livenessProbe:
        exec:
          command: ["cat", "/tmp/healthcheck"]
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

```yaml
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

```yaml
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

* Policies: **Always** (default),  **OnFailure** , **Never**

```yaml
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

```yaml
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

```yaml
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

### Multi-Container Pods

* Multiple containers can share Pod resources (network namespace, volumes).
* Best practice: keep one main container per Pod; add sidecars only when needed.
* Containers in the same Pod can talk over `localhost`.

```yaml
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

### Container Initialization

* **Init containers** run to completion before app containers start.
* Useful for setup tasks, waiting on dependencies, or preconditions.

```yaml
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
        - sh
        - -c
        - >
          until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local;
          do echo waiting for myservice; sleep 5; done
    - name: init-mydb
      image: busybox:1.28
      command:
        - sh
        - -c
        - >
          until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local;
          do echo waiting for mydb; sleep 5; done
```

# Section 19 - Pods Allocation in Kubernetes

### K8s Pod Scheduling

* The **scheduler** runs on the control plane and places Pods onto nodes based on:
  * Resource requests vs available node resources
  * Scheduling constraints and policies
  * `nodeSelector`,  **affinity** , and **anti-affinity**
* **nodeSelector**
  * Defined in Pod spec
  * Manually selects nodes via labels or exact node name

```bash
# Inspect labels on nodes
kubectl get nodes --show-labels

# Example: add a label to a node
kubectl label nodes mycluster-m03 disktype=ssd
```

**Node selector example**

```yaml
# nodeselector.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodeselector
spec:
  containers:
    - name: nginx
      image: nginx
  nodeSelector:
    disktype: ssd
```

**Pin to a specific node by name**

```yaml
# nodename.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodename
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: mycluster-m02
```

**Requests + selector together**

```yaml
# resource-request.yml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-app
spec:
  containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
      resources:
        requests:
          memory: 64Mi
          cpu: 1000m
  nodeSelector:
    disktype: ssd
```

### DaemonSets

* Ensures **one Pod per node** (optionally per matching subset of nodes)
* New nodes get the Pod automatically
* Common use cases: monitoring agents, log collectors, host-level proxies

```yaml
# daemonset.yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logging
spec:
  selector:
    matchLabels:
      app: httpd-logging
  template:
    metadata:
      labels:
        app: httpd-logging
    spec:
      containers:
        - name: webserver
          image: httpd
          ports:
            - containerPort: 80
```

> Result: a `logging` Pod runs on each node.

### Static Pods and Mirror Pods

* **Static Pods**
  * Managed directly by **kubelet** on a node
  * API server is not required to create them
  * Kubelet watches a **manifests directory** and ensures Pods listed there are running
* **Mirror Pods**
  * For each static Pod, the kubelet creates a read-only “mirror” object in the API so you can **view** it with kubectl
  * You cannot update/delete static Pods via the mirror

```bash
# On a node (often a control-plane node):
cd /etc/kubernetes/manifests/
# Place any valid Pod YAML here; kubelet will start it as a static Pod
# Then, from the master, verify you can see a mirror Pod with:
kubectl get pods -A
```

### Node Affinity and Anti-Affinity

* More expressive version of `nodeSelector`
* **requiredDuringSchedulingIgnoredDuringExecution** = hard requirement
* **preferredDuringSchedulingIgnoredDuringExecution** = soft preference
* Operators: `In` (affinity), `NotIn` (anti-affinity), etc.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodeaffinity
spec:
  containers:
    - name: nginx
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values: ["ssd"]
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-anti-affinity
spec:
  containers:
    - name: nginx
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: NotIn
                values: ["ssd"]
```

---

# Section 20 - Deployments in Kubernetes

### Scaling Applications

* **RPM** = requests per minute (traffic)
* **Horizontal scaling** : more Pod replicas
* **Vertical scaling** : more CPU/memory per Pod
* **Stateless apps** : no client state; easy to scale horizontally
* **Stateful apps** : keep client state; often scale vertically or use StatefulSets

### ReplicationController (legacy)

* Specifies number of Pod replicas
* Largely superseded by **ReplicaSet** and **Deployment**

```bash
mkdir deployments && cd deployments
vi replication-controller.yml
```

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: alipne-box-replicationcontroller
spec:
  replicas: 3
  selector:
    app: alipne-box
  template:
    metadata:
      name: alpine
      labels:
        app: alipne-box
    spec:
      containers:
        - name: alpine-box
          image: alpine
          command: ["sleep", "3600"]
```

```bash
kubectl apply -f replication-controller.yml
kubectl get replicationcontroller/alipne-box-replicationcontroller
kubectl get pods -o wide
kubectl delete pod alipne-box-replicationcontroller-4p6l4
kubectl scale --replicas=6 replicationcontroller/alipne-box-replicationcontroller
kubectl scale --replicas=2 replicationcontroller/alipne-box-replicationcontroller
kubectl delete -f replication-controller.yml
```

### ReplicaSets

* Modern replacement for ReplicationController
* Supports  **set-based selectors** : `In`, `NotIn`, `Exists`
* Can adopt **bare Pods** that match its selector

```bash
vi replica-set.yml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicas
  labels:
    app: myapp
    tier: frontend
spec:
  replicas: 3
  selector:
    matchExpressions:
      - { key: tier, operator: In, values: [frontend] }
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f replica-set.yml
kubectl get rs/myapp-replicas
```

```bash
vi replicaSet_and_barePods.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod1
  labels:
    tier: frontend
spec:
  containers:
    - name: application1
      image: gcr.io/google-samples/hello-app:1.0
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod2
  labels:
    tier: frontend
spec:
  containers:
    - name: application2
      image: gcr.io/google-samples/hello-app:2.0
```

```bash
kubectl apply -f replicaSet_and_barePods.yml
kubectl get pods -o wide
```

> Because these bare Pods match the ReplicaSet’s selector, the ReplicaSet can manage them.

### Deployments

* High-level controller that manages **ReplicaSets** and **Pods**
* Supports create, update,  **rolling updates** , rollback, pause/resume

```bash
vi deployments.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chef-server
  labels:
    app: chef
spec:
  replicas: 3
  selector:
    matchLabels:
      app: chef-server
  template:
    metadata:
      labels:
        app: chef-server
    spec:
      containers:
        - name: chef-server
          image: "chef/chefdk:4.9"
          ports:
            - containerPort: 8080
          command: ["/bin/sh"]
          args: ["-c", "echo Hello from the Chef container; sleep 3600"]
        - name: ubuntu
          image: "ubuntu:18.04"
          ports:
            - containerPort: 8080
          command: ["/bin/sh"]
          args: ["-c", "echo Hello from the Ubantu container; sleep 3600"]
```

```bash
kubectl apply -f deployments.yml

kubectl get deployment.apps/chef-server
kubectl get pods -o wide
kubectl rollout status deployment.apps/chef-server
kubectl describe deployment.apps/chef-server

kubectl get rs
kubectl get pods --show-labels

kubectl set image deployment/chef-server chef-server=chef/chefdk:4.9.10
kubectl rollout status deployment.apps/chef-server
kubectl get pods --show-labels
kubectl rollout history deployment.apps/chef-server

# record change in history
kubectl set image deployment/chef-server chef-server=chef/chefdk:4.9.14 --record

# undo latest change
kubectl rollout undo deployment.apps/chef-server

# rollback to a specific revision
kubectl rollout undo deployment.apps/chef-server --to-revision=1

# pause, make multiple changes, then resume
kubectl rollout pause deployment.apps/chef-server
kubectl set resource deployment/chef-server -c=chef-server --limits=memory=250Mi
kubectl rollout resume deployment.apps/chef-server

# scale replicas
kubectl scale deployments.apps/chef-server --replicas=5
```

---

# Section 21 - Kubernetes Networking

### Overview

* **Pod-to-Pod networking** across nodes without NAT
* Node agents (like kubelet) can reach all Pods on that node
* Every Pod has its **own IP**

### CNI Plugins

* Pluggable Kubernetes networking via **CNI**
* Multiple choices; pick based on features and docs
* Example: **Calico** works well with kubeadm
* Nodes are **NotReady** until a CNI is installed

### DNS in Kubernetes

* Cluster DNS lets Pods discover other Pods/Services by name
* Common FQDN pattern:
  * Pod: `pod-ip.namespace.pod.cluster.local`
  * Service: `service.namespace.svc.cluster.local`

```bash
mkdir networking && cd networking
vi pods-dns.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodename
spec:
  containers:
    - name: nginx
      image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend-app
spec:
  containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
```

```bash
kubectl get pods -o wide    # note the nginx Pod IP
kubectl exec -it frontend-app -- sh
apk update && apk add curl
curl 10-244-1-15.default.pod.cluster.local
exit
```

### Network Policies

* Control allowed **ingress** and **egress** traffic
* Identify peers by Pod labels, namespaces, or IP blocks
* By default, Pods are **non-isolated**
* Use `podSelector` to select target Pods in a namespace; empty selects all Pods
* Apply to ingress, egress, or both

> Note: Some labs won’t behave on plain Minikube. A kubeadm/HA cluster with a policy-aware CNI is recommended.

```bash
vi network-pol-pods.yml
kubectl create namespace network-policy
kubectl label namespace network-policy role=test-network-policy
kubectl get namespaces --show-labels
kubectl apply -f network-pol-pods.yml
kubectl get pods -o wide -n network-policy
kubectl exec -n network-policy busybox-pod -- curl 10.244.2.3   # IP of nginx Pod
```

```bash
vi network-policy.yml
kubectl apply -f network-policy.yml
kubectl get networkpolicy -n network-policy -o wide
kubectl exec -n network-policy busybox-pod -- curl 10.244.2.3

# Update policy and re-apply
vi network-policy.yml
kubectl apply -f network-policy.yml
kubectl get networkpolicy -n network-policy -o wide
kubectl exec -n network-policy busybox-pod -- curl 10.244.2.3
```

---

# Section 22 - Kubernetes Services

### Services Overview

* Stable access to a set of Pods
* Abstracts Pods behind a virtual IP and label selector
* **Service routing** : client → Service → load-balanced to matching Pod endpoints
* **Endpoints** : the actual Pod IP:port targets behind a Service

### Service Types

* **ClusterIP** (default): reachable inside the cluster only
* **NodePort** : exposes the Service on each node’s IP at a static port
* **LoadBalancer** : provisions external LB (cloud)
* **ExternalName** : returns a CNAME record

```bash
mkdir services_k8s
cd services_k8s/
vi spc-pod.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-server
  labels:
    app: frontend
spec:
  replicas: 3
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
          image: nginx
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f spc-pod.yml
kubectl describe deployment.apps/nginx-server
kubectl get pods
```

```bash
vi clusterip-svc.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080   # adjust to your container’s listening port
```

```bash
kubectl apply -f clusterip-svc.yml
kubectl describe service/nginx-service

# This won't work from outside the cluster:
curl nginx-service:8080
```

Create a test Pod to curl the Service from inside the cluster:

```bash
vi temp-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-svc-test
spec:
  containers:
    - name: busybox
      image: radial/busyboxplus:curl
      command: ["sleep", "3600"]
```

```bash
kubectl apply -f temp-pod.yml
kubectl exec pod-svc-test -- curl nginx-service:8080
```

**NodePort Service**

```bash
vi nodeport-svc.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-nodeport
spec:
  type: NodePort 
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30099
```

```bash
kubectl apply -f nodeport-svc.yml
kubectl describe service/nginx-service-nodeport
curl localhost:30099
```

### Discover Services via DNS

* Kubernetes DNS assigns names to Services:
  * FQDN: `service.namespace.svc.cluster.local`
  * Within same namespace, you can just use `service`

```bash
vi dns-service-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: svc-test-dns
  namespace: service-namespace
spec:
  containers:
    - name: busybox-svc
      image: radial/busyboxplus:curl
      command: ["sleep", "3600"]
```

### Manage Access via Ingress

* Ingress manages external access to in-cluster Services
* Features: SSL termination, load balancing, name-based virtual hosts
* Requires an **ingress controller** running in the cluster

```bash
vi nginx-deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-official-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-official
  template:
    metadata:
      labels:
        app: nginx-official
    spec:
      containers:
        - name: nginx-official
          image: "nginx:latest"
          ports:
            - containerPort: 8080
```

```bash
vi nginx-deployment-service.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-official-service
spec:
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      nodePort: 31303
  selector:
    app: nginx-official
```

```bash
vi magicalnginx-deployment-service.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: magical-nginx
spec:
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      nodePort: 31304
      name: http
  selector:
    app: magical-nginx
```

```bash
vi magicalnginx-nginx-deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: magicalnginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: magical-nginx
  template:
    metadata:
      labels:
        app: magical-nginx
    spec:
      containers:
        - name: magical-nginx
          image: "anshuldevops/magicalnginx:latest"
          ports:
            - name: nginx-port
              containerPort: 3000
```

```bash
vi ingress-controller.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-rules
spec:
  rules:
    - host: nginx-official.example.com
      http:
        paths:
          - path: /
            pathType: Exact
            backend:
              service:
                name: nginx-official-service
                port:
                  number: 80
    - host: magical-nginx.example.com
      http:
        paths:
          - path: /
            pathType: Exact
            backend:
              service:
                name: magical-nginx
                port:
                  number: 80
```

> Ensure an ingress controller (e.g., NGINX Ingress Controller) is installed; otherwise the Ingress resource won’t route traffic.

# Section 23 - Kubernetes Storage

### Storage Overview

* **Container filesystem is ephemeral.** Anything written inside a container’s root filesystem disappears when the container is recreated.
* **Persistent data** lives outside the container so you don’t lose it when Pods restart or move.
* **Volumes** are how Pods access storage at runtime.
* **PersistentVolume (PV)** turns storage into a first-class Kubernetes resource that Pods can claim and use.
* **Common volume types**
  * **NFS** (network file shares)
  * **Cloud disks** (EBS, Persistent Disk, etc.)
  * **ConfigMaps/Secrets** (config as files)
  * **Node filesystem** (e.g., `hostPath`)

> Mental model: the **Pod** brings the app, the **Volume** brings the data. A **PVC** is the app asking for storage; a **PV** is the actual piece of storage that satisfies the ask.

---

### Using Kubernetes Volumes

* You define a **volume** at the  **Pod level** .
* You attach it to containers via **volumeMounts** at the  **container level** .

#### `emptyDir` (scratch space for a Pod)

* Created when a Pod is scheduled to a node.
* Lives for the **lifetime of the Pod** (if the Pod is deleted or rescheduled, data is gone).
* All containers in the Pod can share it.

#### Sharing a volume between containers (sidecar pattern)

* Multiple containers in the same Pod can mount the same volume at different paths and read/write the same files.
* Useful for producer/consumer or “generate files here, serve them there” patterns.

---

### `hostPath` volume (node’s local disk)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  volumes:
    - name: hostpath-vol
      hostPath:
        path: /var/tmp
  containers:
    - name: hostpath-pod
      image: 'k8s.gcr.io/busybox'
      command: ["/bin/sh", "-c", "echo Hello Team, This is Sample File for HostVolume - $(date) >> /output/output.txt"]
      volumeMounts:
        - name: hostpath-vol
          mountPath: /output
```

* Writes go to the **node’s** `/var/tmp/output.txt`. If the Pod dies, the file persists on that node.
* Caution: **node-coupling.** If the Pod moves to another node, it won’t see the old data.

---

### `emptyDir` volume (ephemeral scratch)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-emptydir
spec:
  containers:
    - name: redis
      image: redis
      volumeMounts:
        - name: redis-storage
          mountPath: /data/redis
  volumes:
    - name: redis-storage
      emptyDir: {}
```

* Data lasts only as long as the **Pod** lives. Great for caches, temp files, build artifacts.
* Optional: `emptyDir: { medium: "Memory" }` to back it with RAM.

---

### Shared volume between containers in one Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-multi-container
spec:
  volumes:
    - name: html
      emptyDir: {}
  containers:
    - name: nginx-container
      image: nginx
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
    - name: debian-container
      image: debian
      volumeMounts:
        - name: html
          mountPath: /html
      command: ["/bin/sh", "-c"]
      args:
        - while true; do date >> /html/index.html; sleep 5; done
```

* The **Debian** container writes to `/html/index.html`.
* The **Nginx** container serves the **same file** from `/usr/share/nginx/html`.
* Because they share the  **same volume** , changes are visible instantly.

> Mental model: one container “bakes the cake,” the other “serves the cake,” both using the same kitchen shelf (the shared volume).

---

### Persistent Volumes (PV), Claims (PVC), and StorageClasses

* **PersistentVolume (PV):** a chunk of storage managed by the cluster (could be local disk, NFS, cloud disk).
* **PersistentVolumeClaim (PVC):** a request for storage from a Pod (“I need 200Mi, ReadWriteOnce”).
* **Binding:** Kubernetes binds a PVC to a suitable PV that meets the request.
* **StorageClass:** templates that describe *how to provision* storage and with what parameters.
  * `allowVolumeExpansion: true` lets you resize volumes (if the underlying storage supports it).
  * `volumeBindingMode: WaitForFirstConsumer` delays PV binding until the Pod is scheduled, so the storage can be created in the right zone/node.

#### StorageClass (no external provisioner)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

* `no-provisioner` means **manual** or **pre-provisioned** PVs only (no dynamic provisioning).

#### PersistentVolume (PV)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-persistnt-vol
spec:
  storageClassName: local-storage
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/tmp
```

* **Reclaim policy**
  * **Retain:** keep data after PVC is deleted
  * **Delete:** delete underlying storage with the PV
  * **Recycle:** basic scrub of data (note: deprecated in many environments, but shown here as in your notes)
* **Access modes**
  * **ReadWriteOnce (RWO):** mounted read-write by a single node
  * **ReadOnlyMany (ROX):** mounted read-only by many nodes
  * **ReadWriteMany (RWX):** mounted read-write by many nodes

```bash
kubectl get pv -o wide
```

#### PersistentVolumeClaim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

* Requests **100Mi** of storage (note the capital **M** in Mi).
* With `WaitForFirstConsumer`, binding may show “Pending” until a Pod that uses it is scheduled.

```bash
kubectl get pvc -o wide   # Status may wait for the first consumer (Pod)
```

#### Using your PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pv-pod
spec:
  restartPolicy: Never
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo Hello Team, This is Persistnent Volume Claim >> /output/success.txt"]
      volumeMounts:
        - mountPath: /output
          name: my-pv
  volumes:
    - name: my-pv
      persistentVolumeClaim:
        claimName: my-pvc
```

* The Pod writes to `/output/success.txt`, which is stored on the  **PV** .
* Delete and recreate the Pod, and the file will still be there, because the data lives with the  **PV** , not the container.

#### Expanding a PVC

```bash
kubectl edit pvc my-pvc   # change 100Mi to 200Mi (if StorageClass allows expansion)
```

* Expansion succeeds only if:
  * `allowVolumeExpansion: true` on the StorageClass, and
  * The underlying storage supports resizing.
* Some filesystems require the **Pod** to restart to see the new size.

---

### Quick decision guide

* Use **`emptyDir`** for temporary scratch data or cross-container sharing in a single Pod.
* Use **`hostPath`** only for node-local experiments or when you truly need node files; it ties you to a node.
* Use **PV/PVC** (with a  **StorageClass** ) for real persistence that survives Pod rescheduling and node rotations.
* Pick **access modes** and **reclaim policy** based on how your app reads/writes and how you want to handle lifecycle.

> TL;DR: `emptyDir` is your notepad, `hostPath` is a drawer in one desk, **PV/PVC** is proper storage in the building that you can claim, release, and move workers around without losing files.

# Section 24 - Self Managed K8s on GCP

**Markdown**

```
### Spin K8s Self Managed Cluster on GCP: 2025 Edition

Here is a complete, updated guide to replace the outdated instructions. This guide includes all the fixes and best practices we discovered, such as using a larger master node, configuring the `containerd` runtime correctly, and using modern package repositories.

### 1. Set Project and Zone in Google Cloud

These commands configure your Cloud Shell environment to work with the correct project and a specific geographic zone.

```bash
# Replace <myProject> with your actual GCP Project ID
gcloud config set project <myProject>

gcloud config set compute/zone us-east1-b
```

### 2. Create the VPC Network and Subnet

A Virtual Private Cloud (VPC) provides a private network for your VMs. We create the main network and then a specific subnet with an IP range for our nodes.

**Bash**

```
# Create the VPC
gcloud compute networks create k8s-cluster --subnet-mode custom

# Create the subnet in the correct region for our zone
gcloud compute networks subnets create k8s-nodes \
  --network k8s-cluster \
  --range 10.240.0.0/24 \
  --region us-east1
```

### 3. Create Firewall Rules

These rules control traffic to and from your VMs. We need to allow internal cluster communication, as well as external access for SSH and the Kubernetes API.

**Bash**

```
# Allow internal communication between nodes on all protocols
gcloud compute firewall-rules create k8s-cluster-allow-internal \
  --allow tcp,udp,icmp,ipip \
  --network k8s-cluster \
  --source-ranges 10.240.0.0/24

# Allow external traffic for SSH (port 22) and the Kubernetes API (port 6443)
gcloud compute firewall-rules create k8s-cluster-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network k8s-cluster \
  --source-ranges 0.0.0.0/0
```

### 4. Create the Controller (Master) VM

We will create a VM that is powerful enough to run the Kubernetes control plane stably, using an up-to-date Ubuntu LTS image.

**Bash**

```
gcloud compute instances create master-node \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2204-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-4 \
    --private-network-ip 10.240.0.11 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet k8s-nodes \
    --zone us-east1-b \
    --tags k8s-cluster,master-node,controller
```

### 5. Create the Worker VMs

We will create two smaller worker nodes using an up-to-date Ubuntu LTS image.

**Bash**

```
for i in 0 1; do
  gcloud compute instances create workernode-${i} \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2204-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-2 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet k8s-nodes \
    --zone us-east1-b \
    --tags k8s-cluster,worker
done
```

### 6. Prepare All Nodes (Run on Master and Both Workers)

The following steps must be performed on **all three VMs** (`master-node`, `workernode-0`, and `workernode-1`). You will need to SSH into each one to run these commands (`gcloud compute ssh <vm_name>`).

### 6.1. Install and Configure `containerd` Runtime

Kubernetes now uses `containerd` instead of Docker. We need to install it and ensure its cgroup driver is set correctly to avoid critical errors.

**Bash**

```
# Install containerd
sudo apt-get update
sudo apt-get install -y containerd

# Create a default configuration file
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Set the cgroup driver to systemd
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd to apply the new config
sudo systemctl restart containerd
```

### 6.2. Disable Swap

Kubernetes requires that you disable swap memory on all nodes.

**Bash**

```
sudo swapoff -a
# (Optional but recommended) To make this permanent, comment out the swap line in /etc/fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### 6.3. Install Kubernetes Packages (`kubeadm`, `kubelet`, `kubectl`)

This uses the modern, recommended method for adding the Kubernetes package repositories.

**Bash**

```
# Install prerequisite packages
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Add Kubernetes GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL [https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key](https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key) | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] [https://pkgs.k8s.io/core:/stable:/v1.30/deb/](https://pkgs.k8s.io/core:/stable:/v1.30/deb/) /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install the tools
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 7. Initialize the Control Plane (Run Only on Master Node)

This command bootstraps the cluster. The `--pod-network-cidr` is required for our network plugin (Calico).

**Bash**

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

After this command succeeds, it will print a `kubeadm join` command. **Copy it and save it somewhere safe.**

### 8. Configure `kubectl` (Run Only on Master Node)

This allows you to manage the cluster as a regular user.

**Bash**

```
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 9. Install the Network Plugin (Run Only on Master Node)

This installs Calico, allowing your pods and nodes to communicate.

**Bash**

```
kubectl apply -f [https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml](https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml)
```

### 10. Join Worker Nodes to the Cluster

For each worker node (`workernode-0` and `workernode-1`), run the `kubeadm join` command you saved from Step 7. Remember to run it with `sudo`.

**Bash**

```
# (Run this on each worker node)
sudo kubeadm join 10.240.0.11:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### 11. Verify the Cluster Status (Run on Master Node)

Check that all nodes have joined and are in the `Ready` state. It may take a minute or two for the worker nodes to become ready after joining.

**Bash**

```
kubectl get nodes -w
```

You should see all three nodes listed with a `STATUS` of `Ready`. Your cluster is now fully operational.

# Section 25 - Troubleshoot Self Managed K8s Cluster

### Troubleshooting K8s Clusters

* If Kube API Server is down, user won't be able to use Kubectl to Interact with Cluster.
* User may get the error message look like -
* "The connection to the server localhost:6443 was refused - did you specify the right host and port?"
* Possible Fixes : Make Sure Docker and Kubelet services are up and running on your master node(s).
* ```
  kubectl get nodes
  sudo systemctl status docker
  kubectl dedscribe node <node-name>
  sudo systemctl status kubelet
  sudo systemctl start kubelet
  sudo systemctl enable kubelet
  kubectl get pods -n kube-system
  kubectl describe pod podname -n kube-system

  ```


### Cluster and Node Logs

```
sudo journalctl -u kubectl
sudo journalctl -u kubectl -n 100
sudo journalctl -u kubelet -f (in worker node)
kubectl logs etcd-master-node -n kube-system
```


### Troubleshoot Applications in K8s

* kubectl exec `<pod-name>` -c `<container-name>` -- command
* Kubectl exec is not used to execute Software in container that is not present in container.

vi deployment.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chef-server
  labels:
    app: chef
spec:
  replicas: 3
  selector:
    matchLabels:
      app: chef-server
  template:
    metadata:
      labels:
        app: chef-server
    spec:
      containers:
        # This container uses a known-good public image
        - name: nginx-server
          image: "nginx:latest"
          ports:
            - containerPort: 80
          command: ["/bin/sh"]
          args: ["-c", "echo Hello from the Nginx container; sleep 3600"]
        # This container does not need to expose a port
        - name: ubuntu
          image: "ubuntu:18.04"
          command: ["/bin/sh"]
          args: ["-c", "echo Hello from the Ubuntu container; sleep 3600"]
```

kubectl apply -f deployment.yml

kubectl exec chef-server-dbc4669b7-6gxgw  -c ubuntu -- ls

kubectl exec -it chef-server-dbc4669b7-6gxgw  -c nginx-server -- ls

kubectl exec -it chef-server-dbc4669b7-6gxgw  -c nginx-server -- /bin/bash


### Container Logs in K8s

kubectl logs chef-server-dbc4669b7-6gxgw -c ubuntu

kubectl logs chef-server-dbc4669b7-6gxgw -cnginx-server
