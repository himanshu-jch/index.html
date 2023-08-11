# <center> <u>Metal LB Setup</u> 

## Table of contents

[1 . Requirement of Metal LB:](#1-requirement-of-metal-lb)

[2 . Definition of Metal LB:](#2-definition-of-metallb)

[3 . Command for the setup or configuration:](#3--command-for-the-setup-or-configuration)  

[4 . Create a configmap in which we will define the range of IPs which we can provide to a service:](#4--create-a-configmap-in-which-we-will-define-the-range-of-ips-which-we-can-provide-to-a-service)

## 1 . Requirement of Metal LB

In the bare metal deployment kubernetes does not provide the functionality of creating Load Balancer on service by default

**Env detail(OS name/os version/docker-or-podman-or base):**

**List of tools and technologies:**

* **Kubernetes**

* **Metal LB**

### 2 . Definition of Metal LB

* **Metal LB**  provides us the functionality of providing External IP or Loadbalancer IP on a service.

* **Kubernetes** is an open-source container orchestration system for automating software deployment, scaling, and management.

### 3 .  Command for the setup or configuration

* **Step 1 :**  create a namespace with name metal lb
	
	```
	kubectl create ns metallb
	```

* **Step 2 :**
	
	```
	kubectl config set-context --current --namespace=metallb
	```

* **Step 3 :**

	```
	helm install metallb metallb/metallb
	```

* **Step 4 :**

	```
	Kubectl get pods -n metallb
	```

![](metal1.png)

### 4 . Create a configmap in which we will define the range of IPs which we can provide to a service : 

* **Step 1 :**  

        # root@k8-master-node01:~# cat cm.yaml

        apiVersion: v1

        kind: ConfigMap

        metadata:
        n ame: config
        data:
        config: |
        address-pools:
        name: default
        protocol: layer2
        addresses:
        192.168.122.50-192.168.122.60

* **Step 2 :**
  
	```
	kubectl create -f cm.yaml
	```

* **Step 3 :**

	```
	Deploy a sample nginx application.
	```

* **Step 4 :**

	```
	kubectl create deploy nginx --image nginx
	```

* **Step 5 :**

	```
	kubectl expose deploy nginx --port 80 --type LoadBalancer
	```

![](metal.png)
