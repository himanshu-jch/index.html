# <center> <u>   MetalLB Setup </u> 

## Table of contents

[1 . Requirement of MetalLB](#1--requirement-of-metallb)

[2 . What is Kubernetes ?](#2--what-is-kubernetes)

[3 . What is MetalLB ?](#3--what-is-metallb)


[4 . Install Kubernetes](#4--install-kubernetes)


[5 . Install MetalLB](#5--install-metallb)


[6 . Configure MetalLB :](#6--configure-metallb)


[7 . Test MetalLB :](#7--test-metallb)

## 1 . Requirement of MetalLB

In the bare metal deployment kubernetes does not provide the functionality of creating Load Balancer on service by default

**Env detail(OS name/os version/docker-or-podman-or base) :**

* **Distributor ID :**	Ubuntu
* **Description :**	Ubuntu 22.04.3 LTS
* **Release :**	22.04
* **Codename :**	jammy

**List of tools and technologies:**

* **Kubernetes**

* **MetalLB**

* **Podman/Docker**

### 2 . What is  Kubernetes ?

Think of Kubernetes as a smart manager for your applications. Imagine you have lots of little workers (containers) who need to run your programs. Kubernetes is like a boss who organizes and manages these workers efficiently. It makes sure your programs are running, scales them when needed, and even replaces them if they fail.   


### 3 . What is  MetalLB ?

Picture MetalLB as a friendly valet for your apps in Kubernetes. Normally, LoadBalancers help direct internet traffic to different services, like websites or apps. MetalLB does this even if you're not on a big cloud service (like Amazon or Google). It helps your Kubernetes apps get the right traffic and lets them talk to the world.



### 4 .  Install Kubernetes

Here's a general outline of the steps you would follow to install Kubernetes on Ubuntu:

* **Step 1 :**   **Update and Upgrade :**

		sudo apt-get update
		sudo apt-get upgrade

* **Step 2 :** **Install Docker :**
Kubernetes relies on Docker for containerization. Install Docker using the following commands:

		sudo apt-get install docker.io
		sudo systemctl enable docker
		sudo systemctl start docker

* **Step 3 :** **Install kubeadm, kubelet, and kubectl :**
These are the essential components of Kubernetes.

		sudo apt-get install -y apt-transport-https curl
		curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
		echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
		sudo apt-get update
		sudo apt-get install -y kubelet kubeadm kubectl


  
* **Step 4 :**  **Initialize Kubernetes Master Node (Control Plane) :**
On the master node, you'll initialize Kubernetes using kubeadm. Run the following command to initialize the master node:

		sudo kubeadm init

Follow the instructions provided by the command, including setting up the network pod (CNI) and configuring kubectl for your user.  

* **Step 5 :**  **Configure Kubectl :**
You need to configure kubectl to communicate with the Kubernetes cluster. This is usually done by copying the Kubernetes configuration file into your home directory.

		mkdir -p $HOME/.kube
		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config  
		  
* **Step 6 :** **Install a Network Plugin :**

Kubernetes requires a network plugin for communication between pods across different nodes. A popular choice is Calico:

		kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

* **Step 7 :** **Join Worker Nodes (optional) :**
   
If you have worker nodes, you can join them to the cluster using the token generated during kubeadm init on the master node.   


### 5 .  Install MetalLB
MetalLB is a load balancer implementation for bare metal Kubernetes clusters. It provides a network load balancer implementation for services that use the type LoadBalancer.   

* **Step 1 :**   **Install kubectl (if not already installed) :**
If kubectl is not already installed, you can install it using the following command:

		sudo snap install kubectl --classic


* **Step 2 :**   **Install MetalLB :**
MetalLB can be installed using Kubernetes manifests. Download the MetalLB manifest files using kubectl:

		kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/manifests/namespace.yaml
		kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/manifests/metallb.yaml

### 6 . Configure MetalLB : 

After installing the MetalLB manifests, you need to configure it with a Layer 2 configuration. Create a ConfigMap for MetalLB's configuration:

		apiVersion: v1
		kind: ConfigMap
		metadata:
  		namespace: metallb-system
  		name: config
		data:
  		config: |
    	address-pools:
      - name: default
     	protocol: layer2
     	addresses:
     - <IP_RANGE_START>-<IP_RANGE_END>

Replace **<IP_RANGE_START> and <IP_RANGE_END>** with the range of IP addresses you want MetalLB to allocate from. Save this configuration in a file named metallb-config.yaml and apply it using kubectl:  


		kubectl apply -f metallb-config.yaml

* **Step 4 :**   **Use MetalLB :**  

Now you can create a **Kubernetes Load Balancer service** to use MetalLB. For example:

		apiVersion: v1
		kind: Service
		metadata:
  		name: my-service
		spec:
  		type: LoadBalancer
  		ports:
        - port: 80
    	targetPort: 80
  		selector:
    	app: my-app   

### 7 . Test MetalLB :

Deploy a simple service to test **MetalLB's LoadBalancer functionality.** Create a file named nginx-service.yaml:   


		apiVersion: v1
		kind: Service
		metadata:
  		name: nginx-service
		spec:
  		ports:
        - port: 80
      	targetPort: 80
  		selector:
    	app: nginx-app
  		type: LoadBalancer
  

**Apply the service :**   

		kubectl apply -f nginx-service.yaml

Wait a moment, and then check the service's external IP:

		kubectl get svc nginx-service   

You should see an IP address assigned to the **EXTERNAL-IP** column.

MetalLB is now **set up** in your Kubernetes cluster running on Ubuntu. It's helping to distribute traffic and make your applications accessible through the IP you've configured. 

