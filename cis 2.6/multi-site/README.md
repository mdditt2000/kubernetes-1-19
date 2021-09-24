# ExternalDNS for Multi-Site Kubernetes using F5 CIS with BIG-IP LTM and DNS

ExternalDNS allows user to control DNS records dynamically via Kubernetes CRD resources in a DNS provider-agnostic way. This user-guide documents using F5 CIS with BIG-IP LTM and DNS in a multi-site kubernetes deployment. 

Looking at the diagram below there are two Data Centers **k8s19-cluster** and **k8s20-cluster**. Both Data Center have standalone BIG-IP LTM and DNS **big-ip-60-cluster** and **big-ip-91-cluster**. The BIG-IP DNS devices are synchronized, to share Data Center, Server, Wide IP, and Virtual Server availability. Each Data Centers has Kubernetes deployed with duplicate applications. However Data Center **k8s20-cluster** has a newer version of Kubernetes installed. Creating a second cluster with traffic distribution helps validation of newer kubernetes version, high availability, scaling etc. F5 CIS with BIG-IP LTM and DNS using ExternalDNS solves the multi-site challenges. 

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-23_15-52-41.png)

## Prerequisites

* Recommend AS3 version 3.30 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.30.0)
* CIS 2.6 [repo](coming)
* Clouddocs [documentation](https://clouddocs.f5.com/containers/latest/userguide/crd/externaldns.html)
* Global DNS provisioned and synchronized between Data Centers

## Step 1: Deploy CIS for k8s19-cluster and k8s20-cluster

CIS 2.6 communicates directly with BIG-IP DNS via the Rest API and requires gtm-bigip-username and password. Since BIG-IP LTM and DNS are on the **same BIG-IP** you can re-use the secret generic bigip-login when deploying CIS as shown below.

Add the following parameters to THE CIS deployment

* --custom-resource-mode=true - Configure CIS to only monitor CRDs. CIS will ignore all other resources
* --gtm-bigip-username - Provide username for CIS to access GTM
* --gtm-bigip-password - Provide password for CIS to access GTM
* --gtm-bigip-url - Provide url for CIS to access GTM. CIS uses the python SDK to configure GTM 

Cluster **k8s20-cluster**

```
- args: 
    - "--gtm-bigip-username=$(BIGIP_USERNAME)"
    - "--gtm-bigip-password=$(BIGIP_PASSWORD)"
    - "--gtm-bigip-url=192.168.200.91"
    - "--bigip-username=$(BIGIP_USERNAME)"
    - "--bigip-password=$(BIGIP_PASSWORD)"
    - "--bigip-url=192.168.200.91"
    - "--bigip-partition=k8s"
    - "--pool-member-type=cluster"
    - "--flannel-name=fl-vxlan"
    - "--log-level=DEBUG"
    - "--insecure=true"
    - "--custom-resource-mode=true"
    - "--as3-validation=true"
    - "--log-as3-response=true"
```

Cluster **k8s19-cluster**

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

Deploy CIS in both **k8s19-cluster** and **k8s20-cluster**

```
kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
kubectl create serviceaccount k8s-bigip-ctlr -n kube-system
kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
kubectl create -f f5-cluster-deployment.yaml
kubectl create -f f5-bigip-node.yaml
```
**Note** f5-bigip-node is required for Flannel

* cis-deployment k8s19-cluster [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/multi-site/k8s19-cluster/cis-deployment)
* cis-deployment k8s20-cluster [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/multi-site/k8s20-cluster/cis-deployment)

## Step 2: Deploy F5 Demo App for k8s19-cluster and k8s20-cluster

Deploy the test F5 demo deployment and service. This is a simple application on port 80 and requires a Host Header

```
kubectl create -f pod-deployment
```

* pod-deployment k8s19-cluster [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/multi-site/k8s19-cluster/pod-deployment)
* pod-deployment k8s20-cluster [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/multi-site/k8s20-cluster/pod-deployment)

## Step 3: Create the VirtualServers for k8s19-cluster and k8s20-cluster

**Note** CIS requires the following **Data Center** prerequisites created on **big-ip-60-cluster** BIG-IP DNS for both **k8s19-cluster** amd **k8s20-cluster**

* **DataCenter** with BIG-IP DNS **big-ip-60-cluster** for **k8s19-cluster**

![DataCenter](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-23_16-37-50.png)

* **DataCenter** with BIG-IP DNS **big-ip-60-cluster** for **k8s20-cluster**

![DataCenter](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-23_16-38-45.png)

**Note** CIS requires the following **Data Center** prerequisites created on **big-ip-91-cluster** BIG-IP DNS for both **k8s19-cluster** amd **k8s20-cluster**

* **DataCenter** on with BIG-IP DNS **big-ip-91-cluster** for **k8s19-cluster**

![DataCenter](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-23_16-37-50.png)

* **DataCenter** on with BIG-IP DNS**big-ip-91-cluster** for **k8s20-cluster**

![DataCenter](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-23_16-38-45.png)

**Note** CIS requires the following all **Server** prerequisites created for **big-ip-60-cluster** and **big-ip-91-cluster** BIG-IP DNS for both **k8s19-cluster** amd **k8s20-cluster** as shown in the diagram

![Serverslist](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-37-41.png)

Example server creation for **big-ip-60-cluster** for **k8s19-cluster**

* **Servers** with BIG-IP DNS under GSLB(DNS) by referring:

    - **DataCenter** with **BIG-IP device**

![Servers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-13-22.png)

* **Servers** under GSLB(DNS) by referring:

    - **External SelfIP**

![Servers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-14-53.png)

* **Servers** under GSLB(DNS) by referring:

    - **Virtual Server Discovery enabled**

![Servers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-15-32.png)
    
**Note** Virtual Server Discovery must be enabled for this solution to work. We plan to enhance this in a upcoming release of CIS.

Example server creation for **big-ip-91-cluster** for **k8s19-cluster**

* **Servers** with BIG-IP DNS under GSLB(DNS) by referring:

    - **DataCenter** with **BIG-IP device**

![Servers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-47-43.png)

* **Servers** under GSLB(DNS) by referring:

    - **External SelfIP**

![Servers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-48-22.png)

* **Servers** under GSLB(DNS) by referring:

    - **Virtual Server Discovery enabled**

![Servers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/multi-site/diagrams/2021-09-24_13-49-10.png)


Create the mysite and myapp virtualservers CRDs for both **k8s19-cluster** and **k8s20-cluster**

```
kubectl create -f vs-myapp.yaml
kubectl create -f vs-mysite.yaml
kubectl create -f customresourcedefinitions.yml
```
* crd-resources k8s19-cluster [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/multi-site/k8s19-cluster/crd-example)
* crd-resources k8s20-cluster [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.6/multi-site/k8s20-cluster/crd-example)

Validate both **virtualservers** crd's are created

![virtualservers](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_13-39-20.png)

Connect the **mysite.f5demo.com**

![mysite](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_13-40-14.png)

Connect the **myapp.f5demo.com**

![myapp](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_13-39-58.png)

Verify DataCenter and Server list could learn the new virtualservers LTM in the serverlist

![serverlist](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_13-47-58.png)

Verify the virtualservers created in the servicelist

![serverlist](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_13-50-05.png)

## Step 4: Create the WideIP's using the ExternalDNS CRDs

The diagram below show the **VirtualServer** and **ExternalDNS CRD** used in this user-guide. Important to **Note** the following:

* **Host:** **mysite.f5demo.com** in the **VirtualServer** CRD needs to match **domainName: mysite.f5demo.com** and **pools name: mysite.f5demo.com** **ExternalDNS CRD**
* Use the following string for the **GSLB monitor** in the ExternalDNS CRD

```
 monitor:
      type: http
      send: "GET / HTTP/1.1\r\nHost: mysite.f5demo.com\r\n"
```

* **dataServerName: /Common/big-ip-60-cluster** in the ExternalDNS CRD needs to match DataCenter Server name

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-17_10-25-22.png)

Create the mysite and myapp EDNS CRDs

```
kubectl create -f edns-myapp.yaml
kubectl create -f edns-mysite.yaml
```

Validate the WIDE IP list. You should see both Wide IP created for mysite and myapp and both green status

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-20_15-14-10.png)

If the status for the Wide IP's show red then maybe the external DNS monitor has failed. Check the monitor

![monitor](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-20_15-20-20.png)

Also validate the Send String

![sendstring](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.6/edns/diagram/2021-09-20_15-21-00.png)