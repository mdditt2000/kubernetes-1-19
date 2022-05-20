# Service Type LB with NGINX IC and BIG-IP Firewall/Dos

A service of type LoadBalancer is the simplest and the fastest way to expose a service inside a Kubernetes cluster to the external world. All you need to-do is specify the service type as type=LoadBalancer in the service definition. This guide we are exposing the **nginx-ingress service** as service type type=LoadBalancer.

Services of type LoadBalancer are natively supported in Kubernetes deployments. When you create the **nginx-ingress service** of type LoadBalancer, Kubernetes spins up the service. F5 IPAM Controller allocates an IP address from the ip-range specified by the **ipamlabel**. Using CIS with **nginx-ingress service** configured for type LoadBalancer, BIG-IP can load balance the traffic to **NGINX Ingress Controller** without having to create any ingress resource. CIS will create the public IP addresses on BIG-IP with the **Firewall, DOS and profiles** required to protect the Kubernetes cluster from attacks. This cloud like simplification of load balancer resources could significantly reduce your operational expenses, complexity and prevent network attacks.

With simplicity comes reduced completely or features. Services of type LoadBalancer lacks the capability of advanced configuration. However CIS can solve this with adding a **PolicyCRD** to the Virtual Address that is created on BIG-IP. Ths advanced configuration could be adding **TCP profiles, iRule, SNAT pools, logging, Firewall and DOS profiles etc**. This examples demonstrates adding a firewall policy to protect the Kubernetes cluster from Network attacks. Logging policy is also added to log any attacks

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/servicetypelb_nginx_firewall/diagram/2022-05-17_16-25-35.png)

Demo on YouTube [video]()

## Create the CIS Deployment Configuration

### Step 1

Add the parameter --ipam=true in the CIS deployment to provide the integration with CIS and IPAM

* --ipam=true

```
args: 
  - "--bigip-username=$(BIGIP_USERNAME)"
  - "--bigip-password=$(BIGIP_PASSWORD)"
  - "--bigip-url=192.168.200.60"
  - "--bigip-partition=k8s"
  - "--namespace=default"
  - "--pool-member-type=cluster"
  - "--flannel-name=fl-vxlan"
  - "--log-level=DEBUG"
  - "--insecure=true"
  - "--custom-resource-mode=true"
  - "--as3-validation=true"
  - "--log-as3-response=true"
  - "--ipam=true"
```

Deploy CIS, CRD schema

```
kubectl create -f bigip-ctlr-clusterrole.yaml
kubectl create -f f5-bigip-ctlr-deployment.yaml
kubectl create -f customresourcedefinitions.yaml

```

* cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/servicetypelb_nginx_firewall/cis-deployment)
* crd-schema [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/servicetypelb_nginx_firewall/crd-schema/customresourcedefinitions.yaml)

## F5 IPAM Deployment Configuration

### Step 2

* --orchestration=kubernetes

The orchestration parameter holds the orchestration environment i.e. Kubernetes

* --ip-range='{"Test":"10.192.75.117-10.192.75.119"}'

ip-range parameter holds the IP address ranges and from this range, it creates a pool of IP address range which gets allocated by the **ipamlabel** defined in the Service

* --log-level=debug

```
- args:
    - --orchestration=kubernetes
    - --ip-range='{"Test":"10.192.75.117-10.192.75.119"}'
    - --log-level=DEBUG
```

Deploy RBAC, schema and F5 IPAM Controller deployment

```
kubectl create -f f5-ipam-rbac.yaml
kubectl create -f f5-ipam-persitentvolume.yaml
kubectl create -f f5-ipam-deployment.yaml
```
## Logging output when deploying the F5 IPAM Controller

```
[kube@k8s-1-19-master crd-example]$ kubectl logs -f deploy/f5-ipam-controller -n kube-system
❯ kubectl logs -f deploy/f5-ipam-controller -n kube-system
2022/05/17 23:11:47 [DEBUG] Enqueueing on Update: kube-system/k8s-bigip-ctlr-deployment.k8s.ipam
2022/05/17 23:11:47 [DEBUG] Processing Key: &{0xc0004de580 0xc0000dc2c0 Update}
2022/05/17 23:11:47 [DEBUG] Updated: kube-system/k8s-bigip-ctlr-deployment.k8s.ipam with Status. With IP: 10.192.75.117 for Request:
Hostname:       Key: nginx-ingress/nginx-ingress_svc    IPAMLabel: Test IPAddr:         Operation: Create
```

ipam-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/servicetypelb_nginx_firewall/ipam-deployment)


## Create the PolicyCRD

### Step 3

Create the pod deployments and services for the test application

```
Kubectl create -f policy-type-lb.yaml
```

policy-crd [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/servicetypelb_nginx_firewall/cis-policy)

### Deploy the Cafe Application

**Step 3**

Create the coffee and the tea deployments and services:

    kubectl create -f cafe.yaml

### Configure Load Balancing for the Cafe Application

Create a secret with an SSL certificate and a key:

    kubectl create -f cafe-secret.yaml

Create an Ingress resource:

    kubectl create -f cafe-ingress.yaml

ingress-example [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/servicetypelb_nginx_firewall/ingress-example)


## Create the NGINX Ingress Controller

### Step 5

### Nginx-Controller Installation

This diagram demonstrates the how CIS and IPAM work together using annotations in the **nginx-ingress service** to add Firewall, DOS and other required BIG-IP configurations 

![CRD](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/servicetypelb_nginx_firewall/diagram/2022-05-17_16-47-44.png)

Create NGINX IC custom resource definitions schema:

    kubectl apply -f k8s.nginx.org_virtualservers.yaml
    kubectl apply -f k8s.nginx.org_virtualserverroutes.yaml
    kubectl apply -f k8s.nginx.org_transportservers.yaml
    kubectl apply -f k8s.nginx.org_policies.yaml

crd-schema [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/servicetypelb_nginx_firewall/nginx-config/crd-schema)

Create a namespace and a service account for the Ingress controller:
   
    kubectl apply -f nginx-config/ns-and-sa.yaml
   
Create a cluster role and cluster role binding for the service account:
   
    kubectl apply -f nginx-config/rbac.yaml
   
Create a secret with a TLS certificate and a key for the default server in NGINX:

    kubectl apply -f nginx-config/default-server-secret.yaml
    
Create a config map for customizing NGINX configuration:

    kubectl apply -f nginx-config/nginx-config.yaml
    
Create an IngressClass resource (for Kubernetes >= 1.18):  
    
    kubectl apply -f nginx-config/ingress-class.yaml

Create a deployment for the Ingress Controller pods for ports 80 and 443 as follows: 

    kubectl apply -f nginx-config/nginx-ingress.yaml
  
Create a service for the Ingress Controller pods for ports 80 and 443 as follows: 

Add the annotation to the nginx-service for **LoadBalancer Service type** to obtain the public IP address from IPAM.

```
  annotations:
    cis.f5.com/ipamLabel: Test
    cis.f5.com/policyName: type-lb
    cis.f5.com/health: '{"interval": 10, "timeout": 31}'
```

    kubectl apply -f nginx-config/nginx-service.yaml

nginx-config [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/servicetypelb_nginx_firewall/nginx-config)

## View the Service Type LoadBalancer status

Use the kubectl get service command to determine the EXTERNAL-IP

```
❯ kubectl get service -n nginx-ingress
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
nginx-ingress   LoadBalancer   10.99.152.119   10.192.75.117   80:31203/TCP,443:30561/TCP   106s

```
CIS will add the EXTERNAL-IP to the BIG-IP as you can see in the diagram

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/servicetypelb_nginx_firewall/diagram/2022-05-17_15-12-04.png)

## View the Firewall, DOS and Logging are added to the VirtualServer

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/servicetypelb_nginx_firewall/diagram/2022-05-20_12-16-25.png)

## View the Firewall evening logs

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/servicetypelb_nginx_firewall/diagram/2022-05-20_12-20-56.png)