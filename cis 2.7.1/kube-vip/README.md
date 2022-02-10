# Kube-Vip and F5 BIG-IP with NGINX Ingress Controller

The purpose of this document is to demonstrate Kube-VIP using BGP with NGINX Ingress Controller and F5 BIG-IP. Kube-vip provides Kubernetes clusters with a virtual IP and load balancer for both the control plane (for building a highly-available cluster) and Kubernetes Services of type LoadBalancer using external BIG-IP. This document focuses mainly on the Kubernetes Services of type LoadBalancer. The original purpose of kube-vip was to simplify the building of highly-available (HA) Kubernetes clusters, which at the time involved a few components and configurations that all needed to be managed. Since the project has evolved, it can now use those same technologies to provide load balancing capabilities for Kubernetes Service resources of type LoadBalancer. 

Using type LoadBalancer with NGINX Ingress Controller this allow a virtual IP to load balancer all traffic across the NGINX cluster to the applications pods. This virtual IP can be advertised by BIG-IP using BPG or other routing protocols. This architecture diagram demonstrates the kube-VIP using BGP with NGINX Ingress Controller and F5 BIG-IP

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/kube-vip/diagram/2022-02-10_11-13-59.png)

Demo [YouTube]()

This user-guide demonstrates a virtual IP which is updated by the kube-vip Cloud Provider. Kube-vip Cloud Provider monitors the Nginx-Ingress Service which is of type LoadBalancer and update the EXTERNAL-IP with the virtual IP. This virtual IP can be advertised to the BIG-IP BPG and redistributed to the network. 

## Step 1: Deploy Kube-VIP

In this user-guide, I deployed Kube-VIP as a DaemonSet. The installation documentation can be located **daemonset** [repo](https://kube-vip.chipzoller.dev/docs/installation/daemonset/)

Used the following parameters

* --custom-resource-mode=true - Configure CIS to only monitor CRDs. CIS will ignore all other resources
* --gtm-bigip-username - Provide username for CIS to access GTM
* --gtm-bigip-password - Provide password for CIS to access GTM
* --gtm-bigip-url - Provide url for CIS to access GTM. CIS uses the python SDK to configure GTM
* --ipam=true - CIS to pull VirtualServer IP address from IPAM range

```
- name: bgp_peeras
    value: "65000"
- name: bgp_peers
    value: "192.168.200.61:65000::false"
- name: address
    value: 192.168.200.117
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