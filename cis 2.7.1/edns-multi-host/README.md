# ExternalDNS for NGINX Ingress Controller using F5 CIS with BIG-IP DNS

The purpose of this document is to demonstrate ExternalDNS with NGINX Ingress Controller using F5 CIS with BIG-IP.  ExternalDNS allows user to control DNS records dynamically via Kubernetes CRD resources in a DNS provider-agnostic way. This user-guide documents ExternalDNS with F5 CIS + BIG-IP LTM and DNS, load balancing to NGINX Ingress Controller. BIG-IP LTM and DNS are configured on the same device for a single cluster as shown in the diagram. However BIG-IP LTM and DNS can be on dedicated devices for multiple sites,clusters and data centers. 

ExternalDNS solution provides you with modern, container application workloads that use both BIG-IP Container Ingress Services and NGINX Ingress Controller for Kubernetes. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of BIG-IP and NGINX and fostering better collaboration across NetOps and DevOps teams. The diagram below demonstrates this use-case.

This architecture diagram demonstrates the ExternalDNS with NGINX Ingress Controller using F5 CIS with BIG-IP

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/edns-multi-host/diagram/2022-01-13_10-37-44.png)

[YouTube Demo]()

This user-guide demonstrates a single wide-ip **cafe.example.com** which answers for both coffee and tea deployments. DNS has no layer 7 path awareness and therefore a DNS monitor is required to determine the health of the deployments. Recommendation would be to create a dedicated http status page for the DNS monitor, monitoring required deployments etc. IF the monitor detects the http status failure, the wide-ip is removed from BIG-IP DNS. Another option is to have a 1-1 mapping between the wide-ip and service. 

## Prerequisites

* Recommend AS3 version 3.30 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.30.0)
* F5 CIS 2.7 [repo](https://github.com/F5Networks/k8s-bigip-ctlr/releases/tag/v2.7.0)
* Clouddocs [documentation](https://clouddocs.f5.com/containers/latest/userguide/crd/externaldns.html)

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
    "--disable-teems=true",
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
