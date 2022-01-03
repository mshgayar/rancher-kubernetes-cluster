# rancher-kubernetes-cluster
High Available Kuberentes Cluster with rancher RKE

## Table of contents
* [Cluster Requirements](#cluster-requirements)
* [Tools](#tools)
* [Deployment](#deployment)

## Cluster Requirements
This cluster is going to be deployed on 8 Machines ( virtual or baremetal) & Centos 7 Operating System
* Linux Control Box (Your PC)
	-	( ansible,rke,helm,kubectl) to be installed on this machine
	-	configure ssh key to access all the nodes with pubkey
* All machines have DNS resolution name thorugh DNS or /etc/hosts
* One Load balancer machine (Nginx)
	-	kub-ha.lab.example.com
* Three master nodes
	-	kub-master01.lab.example.com
	-	kub-master02.lab.example.com
	-	kub-master03.lab.example.com
* Three worker nodes
	-	kub-worker01.lab.example.com
	-	kub-worker02.lab.example.com
	-	kub-worker03.lab.example.com
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
```

#### Part 1 ) Installaing Required Tools on Your Linux Control Machine
```
## Install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version


## Install RKE
wget https://github.com/rancher/rke/releases/download/v1.0.0/rke_linux-amd64
chmod +x rke_linux-amd64
sudo mv ./rke_linux-amd64 /usr/local/bin/rke


## Install Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh
sudo chown root: /usr/local/bin/helm
```

#### Part 2 ) Installing the Kubernetes Cluster with rke
Steps :
* configure cluster configuration file 
* Install the kubernetes cluster

``` vim cluster.yml

   cluster_name: mykub
   ssh_key_path: ~/.ssh/cloud-key                                                                                                                                                              
   nodes:
     - address: 192.168.35.140
       user: centos
       role: [controlplane,  etcd, worker]
       hostname_override: kub-master01.lab.example.com
   
    - address: 192.168.35.85
      user: centos
      role: [controlplane,  etcd, worker]
      hostname_override: kub-master02.lab.example.com
  
    - address: 192.168.35.127
      user: centos
      role: [controlplane,  etcd, worker]
      hostname_override: kub-master03.lab.example.com
  
    - address: 192.168.35.165
      user: centos
      role: [worker]
      hostname_override: kub-worker01.lab.example.com
  
    - address: 192.168.35.84
      user: centos
      role: [worker]
      hostname_override: kub-worker02.lab.example.com
  
    - address: 192.168.35.92
      user: centos
      role: [worker]
      hostname_override: kub-worker03.lab.example.com
  
  services:
    etcd:
      snapshot: true
      creation: 12h
      retention: 168h
```
