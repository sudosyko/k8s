sudosyko/k8s
# SA II Project: on Kubernetes Cluster 

## About this Project 

This is the project documentation for the System Administration II course at TSBE. 
This Project covers installing and scaling  on Kubernetes. 

The goal is to install and build a Kubernetes cluster with the following requirements in place:
* scalability through the orchestration with Kubernetes
* containers with persistant storage i.E. a Database
* working container networking with portforwarding for external access
* secure configuration to minimize security risk

The repository also includes the ressources used for this project (check the archive for failed attempts 😉)

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

* minikube


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
```

> Now vmLM1 is accessible over ssh!

### Install & initialy setup Kubernetes on vmLM1

> Install docker as virtualizationlayer for kubernetes
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

> Install Minikube

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

# check status
minikube status

# list addons
minikube addons list

# show cluster info
kubectl cluster-info

# show cluster nodes (should be only one)
kubectl get nodes

kubectl config view

# Enable minikube dashboard
minikube dashboard

# open tunnel to access it from outside (stops working after command is aborted)
kubectl proxy --address='0.0.0.0' --disable-filter=true

# Access Dashboard from: http://192.168.110.60:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/



syntax:                <svc>    <namespace>
kubectl delete service kdash -n kubernetes-dashboard

```


