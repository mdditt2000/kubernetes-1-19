# Per Application Failover using ExternalDNS

The purpose of this document is to demonstrate per application failover using ExternalDNS with NGINX Ingress Controller and F5 BIG-IP. ExternalDNS allows user to control DNS records dynamically via Kubernetes CRD resources in a DNS provider-agnostic way. This user-guide documents ExternalDNS with F5 CIS + BIG-IP LTM and DNS, load balancing to NGINX Ingress Controller. BIG-IP LTM and DNS are configured on the same device for a single cluster as shown in the diagram. However BIG-IP LTM and DNS can be on dedicated devices for multiple sites,clusters and data centers. 

ExternalDNS solution provides you with modern, container application workloads that use both F5 Container Ingress Services and NGINX Ingress Controller for Kubernetes. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of BIG-IP and NGINX and fostering better collaboration across NetOps and DevOps teams. The diagram below demonstrates this use-case.

This architecture diagram demonstrates the ExternalDNS with NGINX Ingress Controller using F5 CIS with BIG-IP for ten applications

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/per-application-failover/diagram/2022-01-31_14-03-56.png)

Demo [YouTube]()

This user-guide demonstrates ten applications, each application having a unique Public Wide IP's **HOST name** which answers for the deployment in Kubernetes. DNS has no layer 7 path awareness and therefore a DNS monitor is required to determine the health of the deployments. Each ExternalDNS CRD would specify the DNS monitor on BIG-IP. Recommended to work with your F5 Solution Architect to discuss DNS monitor scaling. If the monitor detects the http status failure, the Wide IP is removed from BIG-IP DNS.

## Prerequisites

* Recommend AS3 version 3.30 or later [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.30.0)
* F5 CIS 2.7 [repo](https://github.com/F5Networks/k8s-bigip-ctlr/releases/tag/v2.7.0)
* Clouddocs [documentation](https://clouddocs.f5.com/containers/latest/userguide/crd/externaldns.html)

## Step 1: Deploy CIS

CIS 2.7 communicates directly with BIG-IP DNS via the Rest API and requires gtm-bigip-username and password. Since BIG-IP LTM and DNS are on the same device you can re-use the secret generic bigip-login when deploying CIS as shown below. In this example user-guide the Public IP address for BIGIP VirtualServer will be specified by the VirtualServer CRD. 

Add the following parameters to THE CIS deployment

* --custom-resource-mode=true - Configure CIS to only monitor CRDs. CIS will ignore all other resources
* --gtm-bigip-username - Provide username for CIS to access GTM
* --gtm-bigip-password - Provide password for CIS to access GTM
* --gtm-bigip-url - Provide url for CIS to access GTM. CIS uses the python SDK to configure GTM

```
args: [
  # See the k8s-bigip-ctlr documentation for information about
  # all config options
  # https://clouddocs.f5.com/containers/latest/
    "--bigip-username=$(BIGIP_USERNAME)",
    "--bigip-password=$(BIGIP_PASSWORD)",
    "--bigip-url=192.168.200.60",
    "--bigip-partition=k8s",
    "--gtm-bigip-username=$(BIGIP_USERNAME)",
    "--gtm-bigip-password=$(BIGIP_PASSWORD)",
    "--gtm-bigip-url=192.168.200.60",
    "--namespace=nginx-ingress",
    "--pool-member-type=cluster",
    "--flannel-name=fl-vxlan",
    "--insecure=true",
    "--custom-resource-mode=true",
    "--as3-validation=true",
    "--log-as3-response=true",
]
```

Deploy CIS in both locations

```
kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
kubectl create -f bigip-ctlr-clusterrole.yaml
kubectl create -f f5-bigip-ctlr-deployment.yaml
kubectl create -f f5-bigip-node.yaml
```
- f5-bigip-node is required for Flannel
- bigip-ctlr-clusterrole is required for CIS permissions 

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7.1/per-application-failover/cis/cis-deployment)

## Step 3: Nginx-Controller Installation

Deploy NGINX controller by using the following:

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
  
Create a service for the Ingress Controller pods for ports 80 and 443 as follows:

    kubectl apply -f nginx-config/nginx-service.yaml

nginx-config [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7.1/edns-multi-host/nginx-config)

## Step 4: Deploy the Cafe Applications

Create the ten **cafe** deployments and services:

    kubectl create -f cafe.yaml

#### Configure Load Balancing for the Cafe Application

Create a secret with an SSL certificate and a key:

    kubectl create -f cafe-secret.yaml

Create an ten Ingress resource for the **cafe** applications:

```
kubectl create -f brew-ingress.yaml
kubectl create -f chai-ingress.yaml
kubectl create -f coffee-ingress.yaml
kubectl create -f flatwhite-ingress.yaml
kubectl create -f frappuccino-ingress.yaml
kubectl create -f macchiato-ingress.yaml
kubectl create -f mocha-ingress.yaml
kubectl create -f smoothie-ingress.yaml
kubectl create -f tea-ingress.yaml
```

View the Ingress resources for the **cafe** applications:

```
❯ kubectl get ingress
NAME                  CLASS    HOSTS                     ADDRESS   PORTS     AGE
brew-ingress          <none>   brew.example.com                    80, 443   21h
chai-ingress          <none>   chai.example.com                    80, 443   21h
coffee-ingress        <none>   coffee.example.com                  80, 443   3d20h
flatwhite-ingress     <none>   flatwhite.example.com               80, 443   21h
frappuccino-ingress   <none>   frappuccino.example.com             80, 443   21h
latte-ingress         <none>   latte.example.com                   80, 443   3d18h
macchiato-ingress     <none>   macchiato.example.com               80, 443   21h
mocha-ingress         <none>   mocha.example.com                   80, 443   3d18h
smoothie-ingress      <none>   smoothie.example.com                80, 443   21h
tea-ingress           <none>   tea.example.com                     80, 443   3d20h
```

ingress-example [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7.1/per-application-failover/ingress-example)

## Step 4: Create VirtualServer and ExternalDNS CRDs

Create the coffee and the tea VirtualServer CRD

    kubectl create -f vs-tea.yaml
    Kubectl create -f vs-coffee.yaml

Validated VirtualServer on BIG-IP

![VirtualServer](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/edns-multi-host/diagram/2022-01-13_14-29-16.png)

Create the coffee and the tea ExternalDNS CRD

    kubectl create -f edns-tea.yaml
    Kubectl create -f edns-coffee.yaml

Validated VirtualServer CRD and ExternalCRD

**Note** IPAM has provided the external IP address for **cafe.example.com**. **hostGroup: "cafe"** is configured in the VirtualServer CRDs to maintain the same external IP address for all VirtualServer CRDs

```
❯ kubectl get crd,vs,externaldns -n nginx-ingress
NAME                                 HOST               TLSPROFILENAME   HTTPTRAFFIC   IPADDRESS   IPAMLABEL   IPAMVSADDRESS   STATUS   AGE
virtualserver.cis.f5.com/vs-coffee   cafe.example.com   reencrypt-tls    redirect                  Test        10.192.75.117   Ok       6d3h
virtualserver.cis.f5.com/vs-tea      cafe.example.com   reencrypt-tls    redirect                  Test        10.192.75.117   Ok       6d2h

NAME                                 DOMAINNAME         AGE     CREATED ON
externaldns.cis.f5.com/edns-coffee   cafe.example.com   2d15h   2022-01-11T06:36:44Z
externaldns.cis.f5.com/edns-tea      cafe.example.com   27h     2022-01-12T18:57:08Z                                               
```

Validated Wide IP on BIG-IP DNS

![Wide IP](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/edns-multi-host/diagram/2022-01-13_14-20-27.png)