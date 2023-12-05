sudosyko/k8s_awx
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

### Platform & Limitations

The following virtual plattform is hosted by the school to execute the project:

![smart-learn-plan](Docs/Pictures/smart_learn_refference.png)

#### vmKL1
The **vmKL1** is the management machine used to setup, administer and access the kubernetes cluster.
It is a virtual Kali Linux Machine with internet access.

The VM Specs are:
* CPU: 
* RAM:
* Storage:
* Network: Access to vmLM1 & Internet
* IP: 192.168.110.60

#### vmLM11
The **vmLM11** is the Kubernetes host itself. It will act as Control Plane (Master Node) & Data Plane (Worker Node), as we are 

The VM Specs are:
* CPU: 
* RAM: 
* Storage: 
* Network: Access to vmKL1 & Internet
* IP: 192.168.110.60

## What is awx?

AWX is an open-source automation platform that provides a web-based user interface, REST API, and task engine built on top of Ansible.\
Developed by Red Hat, AWX serves as the upstream project for the Ansible Automation Plattform, offering a flexible and scalable solution for managing automation tasks and workflows.

[ansible/awx](https://github.com/ansible/awx)


## What is code-server?

Code-server is an open-source platform that allows developers to run Visual Studio Code (VS Code) in a remote server environment, accessed through a web browser.\
Code-server containers extend this concept further by encapsulating the code-server application and its dependencies within a containerized environment.

[coder/code-server](https://github.com/coder/code-server)

## Setup

### Pre-Reqs vmlKL1:
#### Install useful tools:

> Download the VSCode .deb Package on the following Link:

[VSCode .deb Package](https://go.microsoft.com/fwlink/?LinkID=760868)

> Install following packages:
 * `apt install -y ./<downloaded_vscode_package>.deb`
 * `apt install -y tmux ansible`


### Pre-Reqs vmLM1:
#### open ufw Ports:

> This covers all the necessary ports needed access vmLM1 over SSH
and access kubernetes dashboard, awx and code-server from vmKL1

 * `ufw allow 22, 443, 8443, 10443`
 * `snap install microk8s --classic`



