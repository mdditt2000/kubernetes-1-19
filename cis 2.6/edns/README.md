# ExternalDNS for Kubernetes using F5 CIS with BIG-IP LTM and DNS

ExternalDNS allows user to control DNS records dynamically via Kubernetes CRD resources in a DNS provider-agnostic way. This user-guide documents using F5 CIS with BIG-IP LTM and DNS on the same device for a single cluster as shown in the diagram

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_10-24-26.png)

## Prerequisites

* Recommend AS3 version 3.25 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.30.0)
* CIS 2.2 [repo](coming)
* Clouddocs [documentation](https://clouddocs.f5.com/containers/latest/userguide/crd/externaldns.html)

## Setup Options

CIS 2.6 provides the following options when using ExternalDNS

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

