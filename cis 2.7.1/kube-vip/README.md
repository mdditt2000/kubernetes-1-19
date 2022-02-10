# Kube-Vip and F5 BIG-IP with NGINX Ingress Controller

The purpose of this document is to demonstrate Kube-VIP using BGP with NGINX Ingress Controller and F5 BIG-IP. kube-vip provides Kubernetes clusters with a virtual IP and load balancer for both the control plane (for building a highly-available cluster) and Kubernetes Services of type LoadBalancer using external BIG-IP. This document focuses mainly on the Kubernetes Services of type LoadBalancer. The original purpose of kube-vip was to simplify the building of highly-available (HA) Kubernetes clusters, which at the time involved a few components and configurations that all needed to be managed. Since the project has evolved, it can now use those same technologies to provide load balancing capabilities for Kubernetes Service resources of type LoadBalancer. 

Using type LoadBalancer with NGINX Ingress Controller this allow a virtual IP to load balancer all traffic across the NGINX cluster to the applications. This virtual IP can be advertised by BIG-IP using BPG. This architecture diagram demonstrates the kube-VIP using BGP with NGINX Ingress Controller and F5 BIG-IP

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/kube-vip/diagram/2022-02-10_11-13-59.png)

Demo [YouTube]()

This user-guide demonstrates a single Wide IP **cafe.example.com** which answers for both coffee and tea deployments. DNS has no layer 7 path awareness and therefore a DNS monitor is required to determine the health of the deployments. Recommendation would be to create a dedicated http status page for the DNS monitor, monitoring required deployments etc. IF the monitor detects the http status failure, the Wide IP is removed from BIG-IP DNS. Another option is to have a 1-1 mapping between the Wide IPand service. 

## Step 1: Deploy CIS

CIS 2.7 communicates directly with BIG-IP DNS via the Rest API and requires gtm-bigip-username and password. Since BIG-IP LTM and DNS are on the same device you can re-use the secret generic bigip-login when deploying CIS as shown below. In this example user-guide the Public IP address for BIGIP VirtualServer will be obtained from F5 IPAM. 

Add the following parameters to THE CIS deployment

* --custom-resource-mode=true - Configure CIS to only monitor CRDs. CIS will ignore all other resources
* --gtm-bigip-username - Provide username for CIS to access GTM
* --gtm-bigip-password - Provide password for CIS to access GTM
* --gtm-bigip-url - Provide url for CIS to access GTM. CIS uses the python SDK to configure GTM
* --ipam=true - CIS to pull VirtualServer IP address from IPAM range

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
    "--ipam=true",
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

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7.1/edns-multi-host/cis/cis-deployment)