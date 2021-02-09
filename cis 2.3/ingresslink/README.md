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

## Setup Options

Configure CIS

Create CIS Ingresslink schema

`

`

## Configuration 

* Defining the CIDR label in the virtualserver CRD which maps to the IP-Range. In my example I am using the following 

  - ip range "10.192.75.111/24-10.192.75.115/24"
  - cidr label "10.192.75.0/24"
  - hostname "mysite.f5demo.com" and "myapp.f5demo.com"

* Updating the IP status for the virtualserver CRD

In CIS 2.3 the F5 IPAM Controller can:

* Allocate IP address from static IP address pool based on the CIDR mentioned in a Kubernetes resource

* F5 IPAM Controller decides to allocate the IP from the respective IP address pool for the hostname specified in the virtualserver custom resource

**Note** The idea here is that we will support CRD, Type LB and possibly  route/ingress in the future

