# ExternalDNS for NGINX Ingress Controller using F5 CIS with BIG-IP DNS

The purpose of this document is to demonstrate ExternalDNS with NGINX Ingress Controller using F5 CIS with BIG-IP.  ExternalDNS allows user to control DNS records dynamically via Kubernetes CRD resources in a DNS provider-agnostic way. This user-guide documents ExternalDNS with F5 CIS + BIG-IP LTM and DNS, load balancing to NGINX Ingress Controller. BIG-IP LTM and DNS are configured on the same device for a single cluster as shown in the diagram. However BIG-IP LTM and DNS can be on dedicated devices for multiple sites,clusters and data centers. 

ExternalDNS solution provides you with modern, container application workloads that use both BIG-IP Container Ingress Services and NGINX Ingress Controller for Kubernetes. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of BIG-IP and NGINX and fostering better collaboration across NetOps and DevOps teams. The diagram below demonstrates this use-case.

This architecture diagram demonstrates the ExternalDNS with NGINX Ingress Controller using F5 CIS with BIG-IP

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/edns-multi-host/diagram/2022-01-13_10-37-44.png)