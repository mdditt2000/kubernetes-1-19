# F5 IngressLink

**Note** F5 IngressLink in **PREVIEW**

The F5 IngressLink is addressing modern app delivery at scale/large. IngressLink is a connector between BIG-IP and Nginx using F5 Container Ingress Service and Nginx Ingress Service. The purpose of this page is to documented and simply the configuration and steps required to preview Ingresslink

## IngressLink Compatibility Matrix

Minimum version to use IngressLink:

| CIS | BIGIP | NGINX+ IC | AS3 |
| ------ | ------ | ------ | ------ |
| 2.3+ | v13.1+ | 1.10+ | 3.18+ | 

* Recommend AS3 version 3.25 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.25.0)
* CIS 2.3 [repo](https://github.com/F5Networks/k8s-bigip-ctlr/releases/tag/v2.2.3)
* NGINX+ IC [repo](coming)
* Github [documentation](coming)

## Setup IngressLink in Kubernetes

### Configure CIS component for IngressLink

**Step 1:**

Create CIS Ingresslink CRD schema

```
kubectl create -f ingresslink-customresourcedefinition.yaml
```
cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.3/ipam/crd/big-ip-60-cluster/cis-deployment)

**Step 2:**

* Add the following statements to the CIS deployment arguments for Ingresslink

- "--custom-resource-mode=true"
- "--ingress-link-mode=true"

* In this example I am using ClusterIP mode with VXLAN

- "--pool-member-type=cluster"
- "--flannel-name=fl-vxlan"
          
Deploy CIS 

```
 kubectl create -f f5-cis-deployment.yaml
```

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.3/ipam/crd/big-ip-60-cluster/cis-deployment)