# Simplifying Kubernestes Ingress using F5 Technologies

The purpose of this document is to demonstrate Kubernetes Ingress using F5 BIG-IP and NGINX technologies. This user-guide simplifies the solution by providing examples and step by step guidance. 

F5 Ingress solution provides you with modern, container application workloads that use both BIG-IP Container Ingress Services and NGINX Ingress Controller for Kubernetes. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of BIG-IP and NGINX and fostering better collaboration across NetOps and DevOps teams. The diagram below demonstrates this use-case.

This architecture diagram demonstrates the simplified Kubernetes Ingress solution

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/typelb/diagram/2021-10-15_14-05-41.png)

On this page you’ll find:

* Links to the GitHub repositories for all the requisite software
* Documentation for the solution(s)
* A step by step configuration and deployment guide

## Configure F5 BIG-IP

**Step 1:**

### Install the CIS Controller 

Add BIG-IP credentials as Kubernetes Secrets and create the service account required for deploying CIS

    kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<password>
    kubectl create serviceaccount bigip-ctlr -n kube-system
    kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
    kubectl create -f bigip-ctlr-clusterrole.yaml

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7/typelb/cis/cis-deployment)
    
### Create CIS CRD schema

    kubectl create -f customresourcedefinition.yaml

cis-crd-schema [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/typelb/cis/cis-crd-schema/customresourcedefinition.yaml)

### Update the bigip address, partition and other details(image, imagePullSecrets, etc) in CIS deployment manifest

* Add the following statements to the CIS deployment for CRD support for namespace nginx-ingress

    - "--custom-resource-mode=true"
    - "--namespace=nginx-ingress"

* To deploy the CIS controller in cluster mode update CIS deployment arguments as follows for kubernetes. Additionally, if you are deploying the CIS in Cluster Mode you need to have following prerequisites. For more information, see [Deployment Options](https://clouddocs.f5.com/containers/latest/userguide/config-options.html#config-options)
    
* You must have a fully active/licensed BIG-IP. SDN must be licensed. For more information, see [BIG-IP VE license support for SDN services](https://support.f5.com/csp/article/K26501111).
* VXLAN tunnel should be configured from Kubernetes Cluster to BIG-IP. For more information see, [Creating VXLAN Tunnels](https://clouddocs.f5.com/containers/latest/userguide/cis-helm.html#creating-vxlan-tunnels)

    - "--pool-member-type=cluster"
    - "--flannel-name=fl-vxlan"

* Add the **IPAM parameter** in the CIS deployment to provide **type loadbalance** support

    - "--ipam=true"

```
kubectl create -f f5-cis-deployment.yaml
```

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/typelb/cis/cis-deployment/f5-cis-deployment.yaml)

### Configure BIG-IP as a node in the Kubernetes cluster. This is required for OVN Kubernetes using ClusterIP

    kubectl create -f f5-bigip-node.yaml

bigip-node [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/typelb/cis/cis-deployment/f5-bigip-node.yaml)

**Step 2**

### F5 IPAM Deploy Configuration Options

Add the parameter

* --orchestration=kubernetes - The orchestration parameter holds the orchestration environment i.e. Kubernetes
* --ip-range='{"Test":"10.192.75.113-10.192.75.116"}' - ipamlabel that matches the service. Ranch of public IPs that CIS will configure BIG-IP
* --log-level=debug - recommend info after testing

```
args:
  - --orchestration=kubernetes
  - --ip-range='{"Test":"10.192.75.113-10.192.75.116"}'
  - --log-level=DEBUG
```

Deploy RBAC, Schema, F5 IPAM Controller and Persistent Volumes deployment files

```

kubectl create -f f5-ipam-ctlr-rbac.yaml
kubectl create -f f5-ipam-schema.yaml
kubectl create -f f5-ipam-persitentvolume.yaml
kubectl create -f f5-ipam-deployment.yaml
```
ipam-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7/typelb/ipam-deployment)

**Step 3**

### Nginx-Controller Installation

Create NGINX IC custom resource definitions for VirtualServer and VirtualServerRoute, TransportServer and Policy resources:

    kubectl apply -f k8s.nginx.org_virtualservers.yaml
    kubectl apply -f k8s.nginx.org_virtualserverroutes.yaml
    kubectl apply -f k8s.nginx.org_transportservers.yaml
    kubectl apply -f k8s.nginx.org_policies.yaml

crd-schema [repo](https://github.com/nginxinc/kubernetes-ingress/tree/v1.10.0/deployments/common/crds)

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

Use a Deployment. When you run the Ingress Controller by using a Deployment, by default, Kubernetes will create one Ingress controller pod.

Add the annotation to the nginx-service for service **type=loadbalancer** to obtain the public IP address from IPAM 

```
  annotations:
    cis.f5.com/ipamLabel: Test
```

    kubectl apply -f nginx-config/nginx-ingress.yaml
  
Create a service for the Ingress Controller pods for ports 80 and 443 as follows:

    kubectl apply -f nginx-config/nginx-service.yaml

nginx-config [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7/typelb/nginx-config)

**Step 3**

### Deploy the Cafe Application

Create the coffee and the tea deployments and services:

    kubectl create -f cafe.yaml

### Configure Load Balancing for the Cafe Application

Create a secret with an SSL certificate and a key:

    kubectl create -f cafe-secret.yaml

Create an Ingress resource:

    kubectl create -f cafe-ingress.yaml

ingress-example [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7/typelb/ingress-example)

**Step 4**

### Test the Application