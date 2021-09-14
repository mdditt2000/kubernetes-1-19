# ExternalDNS

ExternalDNS CRD's allows user to control DNS records dynamically via Kubernetes resources in a DNS provider-agnostic way

## Prerequisites

* Recommend AS3 version 3.25 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.25.0)
* CIS 2.2.3 [repo](https://github.com/F5Networks/k8s-bigip-ctlr/releases/tag/v2.2.3)
* Clouddocs [documentation](https://clouddocs.f5.com/containers/latest/userguide/crd/externaldns.html)

This document was created showing the following use cases and topology for ExternalDNS CRD:

Two sites​
Each Site will run an Kubernetes Cluster​
Each Site will have 1 pair of BIG-IPs running LTM and DNS​
Each Kubernetes cluster will run a CIS per BIG-IP (traditional deployment): So 2 CIS will be deployed per site.

## Setup Options

CIS 2.2.3 provides the following options for using CIS

### Step 1

Add the following parameters to both CIS deployment. In this document both LTM and GTM are installed on the same device

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
    - "--namespace=default"
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
kubectl create -f f5-cluster-deployment.yaml
```

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.2.2/ipam/crd/big-ip-60-cluster/cis-deployment)

