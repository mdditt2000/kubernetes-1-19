# ExternalDNS for Kubernetes using F5 CIS with BIG-IP LTM and DNS

ExternalDNS allows user to control DNS records dynamically via Kubernetes CRD resources in a DNS provider-agnostic way. This user-guide documents using F5 CIS with BIG-IP LTM and DNS on the same device for a single cluster as shown in the diagram

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_10-24-26.png)

## Prerequisites

* Recommend AS3 version 3.30 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.30.0)
* CIS 2.6 [repo](coming)
* Clouddocs [documentation](https://clouddocs.f5.com/containers/latest/userguide/crd/externaldns.html)
* Global DNS license under Resource Provisioning Section

## Step 1: Deploy CIS

CIS 2.6 communicates directly with BIG-IP DNS via the Rest API and requires gtm-bigip-username and password. Since BIG-IP LTM and DNS are on the same device you can re-use the secret generic bigip-login when deploying CIS as shown below.

Add the following parameters to THE CIS deployment

* --custom-resource-mode=true - Configure CIS to only monitor CRDs. CIS will ignore all other resources
* --gtm-bigip-username - Provide username for CIS to access GTM
* --gtm-bigip-password - Provide password for CIS to access GTM
* --gtm-bigip-url - Provide url for CIS to access GTM. CIS uses the python SDK to configure GTM 

```
- args: 
    - "--gtm-bigip-username=$(BIGIP_USERNAME)"
    - "--gtm-bigip-password=$(BIGIP_PASSWORD)"
    - "--gtm-bigip-url=192.168.200.60"
    - "--bigip-username=$(BIGIP_USERNAME)"
    - "--bigip-password=$(BIGIP_PASSWORD)"
    - "--bigip-url=192.168.200.60"
    - "--bigip-partition=k8s"
    - "--pool-member-type=cluster"
    - "--flannel-name=fl-vxlan"
    - "--log-level=DEBUG"
    - "--insecure=true"
    - "--custom-resource-mode=true"
    - "--as3-validation=true"
    - "--log-as3-response=true"
```

Deploy CIS in both locations

```
kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
kubectl create serviceaccount k8s-bigip-ctlr -n kube-system
kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
kubectl create -f f5-cluster-deployment.yaml
kubectl create -f f5-bigip-node.yaml
```
**Note** f5-bigip-node is required for Flannel

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/edns/cis-deployment)

## Step 2: Deploy F5 Demo App 

Deploy the test F5 demo deployment and service. This is a simple application on port 80 and requires a Host Header

```
kubectl create -f pod-deployment
```

pod-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/edns/pod-deployment)

## Step 3: Create the VirtualServer CRD and ExternalDNS CRD

CIS requires the following created on DNS

* DataCenter record using the default options

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_10-49-20.png)

* Servers under GSLB(DNS) by referring above DataCenter with BIG-IP device with external SelfIP, Virtual Server Discovery enabled

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_10-52-01.png)