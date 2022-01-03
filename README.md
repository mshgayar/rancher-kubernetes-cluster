# rancher-kubernetes-cluster
High Available Kuberentes Cluster with rancher RKE

## Table of contents
* [Cluster Requirements](#cluster-requirements)
* [Tools](#tools)
* [Deployment](#deployment)

## Cluster Requirements
This cluster is going to be deployed on 8 Machines ( virtual or baremetal) & Centos 7 Operating System
* Linux Control Box (Your PC)
* One Load balancer machine 
* Three master nodes
* Three worker nodes
* Centos 7 OS
	
## Tools
Required tools for the installation :
* Ansible for Nodes preparation (playbooks)
* RKE for kubernetes cluster Installation
* Kubeclt clinet
* Helm chart for rancher deployment
	
## Deployment

```
$ git clone https://github.com/mshgayar/rancher-kubernetes-cluster.git
$ cd rancher-kubernetes-cluster
$ 
