sudosyko/k8s
# SA II Project: wiki.js on Kubernetes Cluster 

## About this Project 

This is the project documentation for the System Administration II course at TSBE. 
This Project covers installing and scaling wiki.js on Kubernetes. 

The goal is to install and build a Kubernetes cluster with the following requirements in place:
* scalability through the orchestration with Kubernetes
* containers with persistant storage i.E. a Database
* working container networking with portforwarding for external access
* secure configuration to minimize security risk

The repository also includes the ressources used for this project (check the archive for failed attempts 😉)

## Kubernetes with Minikube

For this example i built the cluster using minikube as the system ressources provided are very limited.
Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes and it`s installable on Linux, macOS and Windows.
Minikube doesn't have a built-in container engine, but supports various container runtimes, including Docker, containerd, and others. For this project i used docker which is also documented below. The minimal version of docker needs to be 18.09 (20.10 or higher Reccomended)

More requirements for minikube to start:

* 2 CPUs or more.
* 2GB of free memory.
* Internet connection.
* 20GB of free disk space. 

Although this isn`t reccomended, it also worked with 10GB of free disk space, but it performed very bad like this.

[Minikube/Docs](https://minikube.sigs.k8s.io/docs/)

## What is wiki.js ?

Wiki.js is an open-source, modern wiki application that allows you to collaboratively create and edit content in a user-friendly manner. It is designed to be easy to set up and use, making it a popular choice for knowledge management, documentation, and information sharing within teams or organizations. Here are some key features and characteristics of Wiki.js:

* Markdown Support
* Version Control
* Access Control
* User-Friendly Interface
* Search Functionality
* Customization
* Modern Architecture
* Integration and Extensibility
* Multi-Language Support
* Offline Mode
* Security Features

Overall, Wiki.js is a versatile and feature-rich solution for creating and managing documentation and knowledge bases. It is well-suited for teams and organizations that prioritize collaboration, simplicity, and ease of use in their documentation processes.

System Requirements incl. Database:

* 2CPUs
* 1GB RAM
* min. 1GB (Depends on the amount of Data stored on the wiki)

[wiki.js Official Homepage](https://js.wiki/)

**Used Ports:**
* 3000 Frontend
* 3006 MariaDB access

## Platform & Limitations

The following virtual plattform is hosted by the school to execute the project:

![smart-learn-plan](/Docs/Pictures/smart_learn_refference.png)

### vmKL1
The **vmKL1** is the management machine used to setup, administer and access the kubernetes cluster.

The VM Specs are:
* OS: Kali GNU/Linux Rolling (Graphical)
* Kernel: Linux 6.5.0-kali3-amd64
* CPU: 2 vCPUs
* RAM: 16 GB
* Storage: /dev/sda 35 GB 
* Network: Access to vmLM1 & Internet
* IP: 192.168.110.70

Installed Software:

* vscode
* vim
* tmux
* ansible

### vmLM1
The **vmLM1** is the Kubernetes host itself. It will act as Control Plane (Master Node) & Data Plane (Worker Node), as this is the only server provided for the project.

The VM Specs are:
* OS: Ubuntu 22.04.3 LTS
* Kernel: Linux 
* CPU: 4 vCPUs
* RAM: 12 GB
* Storage: /dev/sda 16 GB
* Network: Access to vmKL1 & Internet
* IP: 192.168.110.60

Installed Software:

* minikube version: v1.32.0 // Kubernetes 1.28.3
* docker 
* iptables
* iptables-persitant

## Architecture wiki.js 

This picture gives an overview of the implemented kubernetes architecture for the wiki.js service: 

![k8s_wikijs_architecture](/Docs/Pictures/k8s_wikijs_architecture.png)

## Security & Hardening
The container network is setup in a way that only necessary communication is allowed. UFW & iptables are active and configured.
Open ports to the Kubernetes Node (vmLM1):
* 22/TCP -> SSH
* 443/TCP -> Kubernetes API
* 31650/TCP -> wikijs frontend
* 8001/TCP -> Kubernetes Dashboard
* 8443/TCP -> Docker API

As far as the container network goes, only pods that need external acces are configured with NodePort. all the other pods are configured with Cluster IP for Kubernetes internal only communication.

Also apparmor is active with policies loaded to run kubernetes & docker.
This makes a huge impact on the kernel integrity and thus protects the containers/pods from potential threats.

*Note: the credentials are only shown for demo purposes don´t commit your credentials to a public git repo! also always consider creating secret objects for kubernetes applications as it is unsecure to store credentials directly & unencrpyted in deployment files*

## Manual Setup Guide

### Pre-Requisites vmlKL1:

```bash
# Update the System
sudo apt update -y && apt upgrade -y

# Install the following packages:
sudo apt install -y tmux vim snapd

# Enable & start the snap store
sudo systemctl enable snapd
sudo systemctl start snapd

# Install vscode with snap store
sudo snap install --classic code 
```

Applications installed with the snap store aren´t stored in /usr/bin therefore they are only available over the full path (/snap/bin/code)
>Fix this with the following command:
```bash
sudo ln -s /snap/bin/code /usr/bin/code
```

### Pre-Requisites vmLM1:

```bash
# Enable & open necessary ports on server firewall
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8443/tcp
sudo ufw allow 10443/tcp
sudo ufw enable

# Enable ssh access by uncommenting the following line in /etc/ssh/sshd_config
Port 22

# restart sshd
sudo systemctl restart sshd

# Update the System
sudo apt update -y && apt upgrade -y

# Install iptables-persistent
sudo apt install iptables-persistent

# Reboot the system
```


> Now vmLM1 is accessible over ssh!

### Install docker as virtualizationlayer for kubernetes
```bash

# Install needed packages to sync upstream apt repos
sudo apt install curl wget apt-transport-https -y
sudo apt install ca-certificates curl apt-transport-https

# Install the new gpg keys 
sudo install -m 0755 -d /etc/apt/keyrings

# fetch docker.io gpg keys
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# set access rights to downloaded key
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```bash

# install docker.io apt repository for corresponding os release
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash

# update local apt cache
sudo apt update

# install docker & dependencies
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose

# check status
systemctl status docker

# start & enable docker on startup
sudo systemctl start docker && sudo systemctl enable docker

# set usercontext for vmadmin to use docker
sudo usermod -aG docker $USER
newgrp docker

# check if setting the access rights worked
docker version
```

### Install Minikube

```bash

# Download the installation binary
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install the minikube binary
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install Kubectl utility to manage cluster
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"


chmod +x ./kubectl

# Set bash context for kubectl so we dont have to use the full path
sudo mv kubectl /usr/local/bin/

# check if installation works
kubectl version --client --output=yaml

# start minikube with docker driver
minikube start --vm-driver docker

# Enable minikube dashboard
minikube dashboard

# open tunnel to access it from outside (stops working after command is aborted)
kubectl proxy --address='0.0.0.0' --disable-filter=true

# Access Dashboard from: http://192.168.110.60:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

### wiki.js deployment with external mariadb on minikube

```bash

# Create namespace for wikijs
kubectl create namespace wikijs

# Create project folder for deployment
mkdir ~/projects/wiki-js

# Create data path for persistant storage
sudo mkdir /var/wikijs
```
#### Create hashed credentials for the deployment

```bash
# MySQL root user
echo -n 'root' | base64
# cm9vdA==

# MySQL root user password
echo -n 'password' | base64
# cGFzc3dvcmQ=


# Wiki.js MySQL database name
echo -n 'wikijs' | base64
# d2lraWpz

# Wiki.js MySQL user
echo -n 'wikijs' | base64
# d2lraWpz

# Wiki.js MySQL user Password
echo -n 'password' | base64
# cGFzc3dvcmQ=
```

#### Setup the secret file for the wikijs namespace
> $ vim wikijs-secret.yaml #files are also in wikijs_k8s/
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-secret
  namespace: wikijs
type: Opaque
data:
  ROOT: cm9vdA==
  ROOT_PASSWORD: cGFzc3dvcmQ=
  DATABASE: d2lraWpz
  USER: d2lraWpz
  PASSWORD: cGFzc3dvcmQ=
```
```bash
kubectl apply -f wikijs-secret.yaml
```
#### mariadb & wikijs config setup for k8s
> $ vim wikijs-config.yaml
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: wikijs
spec:
  selector:
    app: mariadb
  ports:
  - name: mariadb
    protocol: TCP
    port: 3306
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: wikijs
  labels:
    app: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: PASSWORD
        - name: MARIADB_ROOT_HOST
          value: "%"
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mariadb-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-storage
        hostPath:
          path: /var/wikijs
          type: DirectoryOrCreate
```
```bash
kubectl apply -f wikijs-config.yaml -n wikijs
```

#### mariadb & wikijs deployment setup for k8s
> $ vim wikijs-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wikijs
  namespace: wikijs
  labels:
    app: wikijs
spec:
  selector:
    matchLabels:
      app: wikijs
  template:
    metadata:
      labels:
        app: wikijs
    spec:
      containers:
      - name: wikijs
        image: requarks/wiki:latest
        imagePullPolicy: Always
        env:
        - name: DB_TYPE
          value: "mariadb"
        - name: DB_HOST
          value: "mariadb"
        - name: DB_PORT
          value: "3306"
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: DATABASE
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: USER
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: PASSWORD
        ports:
        - containerPort: 3000
          name: http
```
```bash
kubectl apply -f wikijs-deployment.yaml -n wikijs
```

#### setup wikijs service for k8s
> $ vim wikijs-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: "wikijs"
  namespace: wikijs
spec:
  type: NodePort
  ports:
    - name: http
      port: 3000
  selector:
    app: "wikijs"

```
```bash
kubectl apply -f wikijs-service.yaml
```
#### Setup Portforwarding & allow Firewall communication
```bash
# Get exposed service port for wikijs
kubectl get svc -n wikijs
# 31650/tcp in our case

# Change: DEFAULT_FORWARD_POLICY="DENY" to DEFAULT_FORWARD_POLICY="ACCEPT"
sudo vim /etc/default/ufw

# Reload UFW
sudo disable ufw
sudo enable ufw

# open the port on ufw
sudo ufw allow 31650/tcp

# uncomment the following line:
# net.ipv4.ip_forward=1
sudo vim /etc/sysctl.conf 

# apply changes:
sudo sysctl -p 

# setup portforwarding for external access (access from vmKL1)
sudo iptables -t nat -A PREROUTING -p tcp --dport 31650 -j DNAT --to-destination 192.168.49.2:31650
```
> Now wikijs is available from vmKL1! 


## Verify installation & useful commands
```bash
# wikijs web secrets
# user: vmadmin@projekt.local
# pw: password

# Manage Runtime of Minikube
minikube status
microk8s stop
microk8s start

# Delete the micork8s cluster
microk8s delete

# list addons
minikube addons list

# show cluster info
kubectl cluster-info

# show cluster nodes (should be only one)
kubectl get nodes

# Show Cluster Config (YAML)
kubectl config view

# Show all Cluster ressources in all namespaces
kubectl get all --all-namespaces

# Show wikijs service
kubectl get svc -n wikijs

# Show wikijs pods
kubectl get pods -n wikijs

# Show wikijs deployment
kubectl get deploy -n wikijs
```
