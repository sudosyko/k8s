sudosyko/k8s_awx

# THIS PROJECT WAS ABORTED DUE TO THE PLATFORM LIMITATIONS!!!!!
Mircok8s pods were stuck in startup loop:
k8s couldnt write to local storage due to other pvc claiming blocking I/O on block level.

![smart-learn-plan](/Docs/Pictures/awx_fail.png)



# SA II Project: awx & code-server on Kubernetes Cluster 
## About this Project 

This is the project documentation for the System Administration II course at TSBE. 
This Project covers installing and scaling awx & code-server on Kubernetes. 

The goal is to install and build a Kubernetes cluster with the following requirements in place:
* scalability through the orchestration with Kubernetes
* containers with persistant storage i.E. a Database
* working container networking with portforwarding for external access
* secure configuration to minimize security risk

The repository also includes the ressources used for this project

## What is awx?

AWX is an open-source automation platform that provides a web-based user interface, REST API, and task engine built on top of Ansible.\
Developed by Red Hat, AWX serves as the upstream project for the Ansible Automation Plattform, offering a flexible and scalable solution for managing automation tasks and workflows.

[ansible/awx](https://github.com/ansible/awx)


## What is code-server?

Code-server is an open-source platform that allows developers to run Visual Studio Code (VS Code) in a remote server environment, accessed through a web browser.\
Code-server containers extend this concept further by encapsulating the code-server application and its dependencies within a containerized environment.

[coder/code-server](https://github.com/coder/code-server)

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

* microk8s


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

```bash
## AS root:
# Install lightweight Kubernetes
snap install --classic microk8s

# Check status of installation & cluster setup
microk8s status --wait-ready

# Alias for ease of use so you dont have to pass microk8s everytime you want to do a kubectl command
alias mkctl="microk8s kubectl"

## AS vmadmin:
# Allow vmadmin to use the microk8s utility & administer the cluster
sudo usermod -a -G microk8s vmadmin
mkdir ~/.kube
cd ~/.kube
sudo microk8s config > config
cd ~
sudo chown -f -R vmadmin ~/.kube
sudo reboot

# Alias for ease of use so you dont have to pass microk8s everytime you want to do a kubectl command
alias kubectl="microk8s kubectl"

# Add alias permanently to your bash shell.
echo 'alias kubectl="microk8s kubectl"' >> ~/.bashrc 

# Set up autocomplete in bash into the current shell, bash-completion package should be installed first.
source <(kubectl completion bash) 

# Add autocomplete permanently to your bash shell.
echo "source <(kubectl completion bash)" >> ~/.bashrc 

# Enable features
microk8s enable dashboard
microk8s enable dns   
microk8s enable ha-cluster 
microk8s enable helm
microk8s enable helm3
microk8s enable metrics-server

# Restart microk8s
microk8s stop
microk8s start

# Alias for ease of use so you dont have to pass microk8s everytime you want to do a helm command
alias helm="microk8s helm"

# Add alias permanently to your bash shell.
echo ' alias helm="microk8s helm"' >> ~/.bashrc 

# View Kubernetes ressources status
kubectl get all --all-namespaces

# Portforward the dashboard to access it from vmKL1
kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443
kubectl get all -n kubernetes-dashboard

```

### Awx deployment on microk8s

```bash
# Add awx-operator repository & install the operator
helm repo add awx-operator https://ansible.github.io/awx-operator/
helm repo update
helm install ansible-awx-operator awx-operator/awx-operator -n awx --create-namespace

# Show the awx pods
sudo kubectl get pods -n awx

# Create the deployment 
mkdir ~/awx-deployment
mkdir /mnt/storage

# Create Deployment Files (see awx-deployment folder for refference)
vim local-storage-class.yaml
vim pv.yaml
vim pvc.yaml
vim ansible-awx.yaml

kubectl create -f local-storage-class.yaml
kubectl get sc -n awx
kubectl create -f pv.yaml
kubectl create -f pvc.yaml
kubectl get pv,pvc -n awx
kubectl create -f ansible-awx.yaml
kubectl get pods -n awx
```

## Install Ansible on vmKL1 & setup ansible management access to vmLM1
This step isn´t mandatory, however it can be useful to have ansible installed on the management machine due to all the utilities that are shiped with it.


```bash
sudo apt install software-properties-common
sudo apt-get -y install ansible
```