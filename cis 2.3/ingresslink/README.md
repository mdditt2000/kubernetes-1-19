# F5 IngressLink

The F5 IngressLink is addressing modern app delivery at scale/large. IngressLink is a connector between BIG-IP and Nginx using F5 Container Ingress Service and Nginx Ingress Service. The purpose of this page is to documented and simply the configuration and steps required to preview Ingresslink

**Currently available as a public preview**,  F5 IngressLink is the first true integration between BIG-IP and NGINX technologies. F5 IngressLink was built to support customers with modern, container application workloads that use both BIG-IP Container Ingress Services and NGINX Ingress Controller for Kubernetes. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of BIG-IP and NGINX and fostering better collaboration across NetOps and DevOps teams. On this page you’ll find:

* Links to the GitHub repositories for all the requisite software
* Documentation for the solution(s)
* A step by step configuration and deployment guide for F5 IngressLink

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

**Step 1:**

### Create the Proxy iRule on Bigip

Proxy Protocol is required by NGINX to provide the applications PODs with the original client IPs. Use the following steps to configure the Proxy_Protocol_iRule

* Login to BigIp GUI 
* On the Main tab, click Local Traffic > iRules.
* Click Create.
* In the Name field, type name as "Proxy_Protocol_iRule".
* In the Definition field, Copy the definition from "Proxy_Protocol_iRule" file. Click Finished.

Proxy_Protocol_iRule [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/ingresslink/big-ip/proxy-protocal/irule)

**Step 2**

### Install the CIS Controller 

Add BIG-IP credentials as Kubernetes Secrets

    kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<password>

Create a service account for deploying CIS.

    kubectl create serviceaccount bigip-ctlr -n kube-system

Create a Cluster Role and Cluster Role Binding on the Kubernetes Cluster as follows:
    
    kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
    
Create IngressLink Custom Resource definition as follows:

    kubectl create -f ingresslink-customresourcedefinition.yaml

cis-crd-schema [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/ingresslink/cis/ingresslink/cis-crd-schema/ingresslink-customresourcedefinition.yaml)

Update the bigip address, partition and other details(image, imagePullSecrets, etc) in CIS deployment file and Install CIS Controller in ClusterIP mode as follows:

* Add the following statements to the CIS deployment arguments for Ingresslink

    - "--custom-resource-mode=true"
    - "--ingress-link-mode=true"

**Note:** To deploy the CIS controller in cluster mode update CIS deploymemt arguments as follows for kubernetes.

    - "--pool-member-type=cluster"
    - "--flannel-name=fl-vxlan"

Additionally, if you are deploying the CIS in Cluster Mode you need to have following prerequisites. For more information, see [Deployment Options](https://clouddocs.f5.com/containers/latest/userguide/config-options.html#config-options)
    
* You must have a fully active/licensed BIG-IP. SDN must be licensed. For more information, see [BIG-IP VE license support for SDN services](https://support.f5.com/csp/article/K26501111).
* VXLAN tunnel should be configured from Kubernetes Cluster to BIG-IP. For more information see, [Creating VXLAN Tunnels](https://clouddocs.f5.com/containers/latest/userguide/cis-helm.html#creating-vxlan-tunnels)
    
Deploy CIS 

    kubectl create -f f5-cis-deployment.yaml

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/ingresslink/cis/ingresslink/cis-deployment/f5-cis-deployment.yaml)

Create bigip-node for VXLAN tunnels

    kubectl create -f f5-bigip-node.yaml

bigip-node [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/ingresslink/cis/ingresslink/cis-deployment/f5-bigip-node.yaml)

Validate that CIS is deployed and running correctly

    [kube@k8s-1-19-master cis-deployment]$ kubectl get pods -n kube-system
    NAME                                                       READY   STATUS    RESTARTS   AGE
    k8s-bigip-ctlr-deployment-fd86c54bb-w6phz                  1/1     Running   0          41s

You can view the CIS logs using the following

**Note** CIS log level is currently set to DEBUG. This can be changed in the CIS controller arguments 

    kubectl logs -f deploy/k8s-bigip-ctlr-deployment -n kube-system | grep --color=auto -i '\[debug'
