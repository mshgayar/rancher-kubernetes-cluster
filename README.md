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
``` kubectl get nodes ```
```
NAME                           STATUS   ROLES                      AGE     VERSION
kub-master01.lab.example.com   Ready    controlplane,etcd,worker   9m59s   v1.16.3
kub-master02.lab.example.com   Ready    controlplane,etcd,worker   9m38s   v1.16.3
kub-master03.lab.example.com   Ready    controlplane,etcd,worker   9m39s   v1.16.3
kub-worker01.lab.example.com   Ready    worker                     9m24s   v1.16.3
kub-worker02.lab.example.com   Ready    worker                     9m23s   v1.16.3
kub-worker03.lab.example.com   Ready    worker                     9m28s   v1.16.3
```
``` kubeclt get pods --all-namespaces ```
```
ingress-nginx   default-http-backend-67cf578fc4-4mq5r     1/1     Running     0          3m35s
ingress-nginx   nginx-ingress-controller-64t7t            1/1     Running     2          3m35s
ingress-nginx   nginx-ingress-controller-jkgr6            1/1     Running     1          3m35s
ingress-nginx   nginx-ingress-controller-k7rgn            1/1     Running     0          3m35s
ingress-nginx   nginx-ingress-controller-vk5hf            1/1     Running     1          3m35s
ingress-nginx   nginx-ingress-controller-zjznp            1/1     Running     0          3m35s
ingress-nginx   nginx-ingress-controller-zkt75            1/1     Running     0          3m35s
kube-system     canal-282hx                               2/2     Running     0          10m
kube-system     canal-blhzc                               2/2     Running     0          10m
kube-system     canal-qfb52                               2/2     Running     0          10m
kube-system     canal-sc6p8                               2/2     Running     2          10m
kube-system     canal-tdzvg                               2/2     Running     1          10m
kube-system     canal-vcnsl                               2/2     Running     2          10m
kube-system     coredns-5c59fd465f-2mrp8                  1/1     Running     0          4m57s
kube-system     coredns-5c59fd465f-7nzlt                  1/1     Running     0          4m33s
kube-system     coredns-autoscaler-d765c8497-g8s55        1/1     Running     0          4m53s
kube-system     metrics-server-64f6dffb84-h86vw           1/1     Running     0          4m6s
kube-system     rke-coredns-addon-deploy-job-pbv4v        0/1     Completed   0          5m8s
kube-system     rke-ingress-controller-deploy-job-5lgh9   0/1     Completed   0          3m59s
kube-system     rke-metrics-addon-deploy-job-q5clr        0/1     Completed   0          4m1s
kube-system     rke-network-plugin-deploy-job-km6d7       0/1     Completed   0          11m
```

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
* Wait for Rancher to be rolled out:
	- 	``` kubectl -n cattle-system rollout status deploy/rancher ```
* ``` kubectl get pods --all-namespace ```
```
NAMESPACE                   NAME                                       READY   STATUS      RESTARTS   AGE
cattle-fleet-local-system   fleet-agent-5f8798bfbf-89g6k               1/1     Running     2          16m
cattle-fleet-system         fleet-controller-df79b5cd8-s5c5x           1/1     Running     1          19m
cattle-fleet-system         gitjob-74cb74b59f-8755b                    1/1     Running     2          19m
cattle-system               helm-operation-2hg69                       0/2     Completed   0          19m
cattle-system               helm-operation-fw2rt                       0/2     Completed   0          18m
cattle-system               helm-operation-j4jm9                       0/2     Completed   0          20m
cattle-system               rancher-7c88dcd6c4-4pv2l                   0/1     Running     5          42m
cattle-system               rancher-7c88dcd6c4-gqq7m                   0/1     Running     3          21m
cattle-system               rancher-7c88dcd6c4-m69hn                   1/1     Running     11         51m
cattle-system               rancher-webhook-6866767fdc-tn8zx           1/1     Running     0          17m
cert-manager                cert-manager-85c9b9bb44-fkk2j              1/1     Running     4          53m
cert-manager                cert-manager-cainjector-78fc9bb777-pllrr   1/1     Running     14         53m
cert-manager                cert-manager-webhook-695f8b56cd-wxfgx      1/1     Running     1          53m
ingress-nginx               default-http-backend-67cf578fc4-4mq5r      1/1     Running     1          61m
ingress-nginx               nginx-ingress-controller-64t7t             1/1     Running     3          61m
ingress-nginx               nginx-ingress-controller-jkgr6             1/1     Running     2          61m
ingress-nginx               nginx-ingress-controller-k7rgn             1/1     Running     1          61m
ingress-nginx               nginx-ingress-controller-vk5hf             1/1     Running     2          61m
ingress-nginx               nginx-ingress-controller-zjznp             1/1     Running     1          61m
ingress-nginx               nginx-ingress-controller-zkt75             1/1     Running     1          61m
kube-system                 canal-282hx                                2/2     Running     2          68m
kube-system                 canal-blhzc                                2/2     Running     2          68m
kube-system                 canal-qfb52                                2/2     Running     2          68m
kube-system                 canal-sc6p8                                2/2     Running     4          68m
kube-system                 canal-tdzvg                                2/2     Running     5          68m
kube-system                 canal-vcnsl                                2/2     Running     5          68m
kube-system                 coredns-5c59fd465f-2mrp8                   1/1     Running     1          62m
kube-system                 coredns-5c59fd465f-7nzlt                   1/1     Running     1          62m
kube-system                 coredns-autoscaler-d765c8497-g8s55         1/1     Running     1          62m
kube-system                 metrics-server-64f6dffb84-h86vw            1/1     Running     1          62m
kube-system                 rke-coredns-addon-deploy-job-pbv4v         0/1     Completed   0          63m
kube-system                 rke-ingress-controller-deploy-job-5lgh9    0/1     Completed   0          62m
kube-system                 rke-metrics-addon-deploy-job-q5clr         0/1     Completed   0          62m
kube-system                 rke-network-plugin-deploy-job-km6d7        0/1     Completed   0          69m
```

## Congrtaualtions : The cluster is up and running
		you can access it through : https://kub-ha.lab.example.com
