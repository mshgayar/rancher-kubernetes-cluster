# rancher-kubernetes-cluster
High Available Kuberentes Cluster with rancher RKE

## Table of contents
* [Cluster Requirements](#cluster-requirements)
* [Tools](#tools)
* [Deployment](#deployment)

## Cluster Requirements
This cluster is going to be deployed on 8 Machines ( virtual or baremetal) & Centos 7 Operating System
* all machines have a user 'centos' 
* Linux Control Box (Your PC)
	-	( ansible,rke,helm,kubectl) to be installed on this machine
	-	configure ssh key to access all the nodes with pubkey
* All machines have DNS resolution name thorugh DNS or /etc/hosts
* One Load balancer machine (Nginx)
	-	kub-ha.lab.example.com
* Three master nodes
	- kub-master01.lab.example.com
	- kub-master02.lab.example.com
	- kub-master03.lab.example.com
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

#### Part 2 )
* prepare the cluster (master and worker nodes ) with ansible playbooks
	-	Installing required packages
	-	disbale firewall & enable selinux
	-	Install the loadbalancer nginx
	-	Installing docker
		
*	```ansible-playbook -i inventory 00-prerequiestes.yml ```
*	```ansible-playbook -i inventory 01-lb-nginx.yaml ```
*	```ansible-playbook -i inventory 02-install-docker.yaml ```


#### Part 3 ) Installing the Kubernetes Cluster with rke
Steps :
* configure cluster configuration file 
* Install the kubernetes cluster

```vim cluster.yml ```
    
Install the kubernetes cluster with rke
```
rke up --config cluster.yml
```
Notes
Save a copy of the following files in a secure location:
 - cluster.yml: 
 	- 	The RKE cluster configuration file.
 - kube_config_cluster.yml: 
 	- 	The Kubeconfig file for the cluster, this file contains credentials for full access to the cluster.
 - cluster.rkestate: 
 	- 	The Kubernetes Cluster State file, this file contains credentials for full access to the cluster.
 - copy kube_config_cluster.yml to $HOME/.kube/config, or if you are working with multiple Kubernetes clusters
	-	``` export KUBECONFIG=$(pwd)/kube_config_cluster.yml ```

Now Kubernetes cluster is ready 
* ``` kubectl get nodes ```
* ``` kubectl get pods --all-namespaces```

#### Part 4 : install rancher platform with helm on the kubernetes cluster
* INSTALL RANCHER HELM CHART
	-	``` helm repo add rancher-stable https://releases.rancher.com/server-charts/stable ```
	-	``` kubectl create namespace cattle-syste ```

*  Install the CustomResourceDefinition resources separately
	-	``` kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.4/cert-manager.crds.yaml```

* Create the namespace for cert-manager
	-	``` kubectl create namespace cert-manager ``` 
*  Add the Jetstack Helm repository
	-	``` helm repo add jetstack https://charts.jetstack.io```
* Update your local Helm chart repository cache
	- ```helm repo update ```
* Install the cert-manager Helm chart
	-	```helm install  cert-manager jetstack/cert-manager --namespace cert-manager  --version v1.0.4 ```
* Once you have installed cert-manager, you can verify it is deployed correctly by checking the cert-manager namespace for running pods:
	-	``` kubectl get pods --namespace cert-manager ```
	-	``` helm install rancher rancher-stable/rancher --namespace cattle-system --set hostname=kub-ha.lab.example.com```
