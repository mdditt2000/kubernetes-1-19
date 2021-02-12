# F5 IPAM Controller for CIS 2.3

**Note** F5 IPAM Controller is currently in **PREVIEW**

The F5 IPAM Controller is a Docker container that allocates IP addresses from an static list for hostnames. The F5 IPAM Controller watches CRD resources and consumes the hostnames within each resource.

## Prerequisites

* Recommend AS3 version 3.25 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.25.0)
* CIS 2.3 [repo](https://github.com/F5Networks/k8s-bigip-ctlr/releases/tag/v2.2.3)
* F5 IPAM Controller [repo](https://github.com/f5devcentral/f5-ipam-controller/releases/tag/v0.1.0)
* Github [documentation](https://github.com/f5devcentral/f5-ipam-controller/blob/main/README.md)

## Setup Options

CIS 2.3 provides the following options for using the F5 IPAM controller

* Defining the CIDR label in the virtualserver CRD which maps to the IP-Range. In my example I am using the following 

  - ip range "10.192.75.111/24-10.192.75.115/24"
  - cidr label "10.192.75.0/24"
  - hostname "mysite.f5demo.com" and "myapp.f5demo.com"

* Updating the IP status for the virtualserver CRD

In CIS 2.3 the F5 IPAM Controller can:

* Allocate IP address from static IP address pool based on the CIDR mentioned in a Kubernetes resource

* F5 IPAM Controller decides to allocate the IP from the respective IP address pool for the hostname specified in the virtualserver custom resource

**Note** The idea here is that we will support CRD, Type LB and possibly  route/ingress in the future

## F5 CIS Configuration Options for IPAM Deployment defining the CIDR network label in the VirtualServer CRD

### Step 1

Add the parameter --ipam=true in the CIS deployment to provide the CIDR parameter in virtualserver CRD

* --ipam=true

```
- args: 
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
    - "--ipam=true"
```

Deploy CIS

```
kubectl create -f f5-cluster-deployment.yaml
```

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.3/ipam/crd/big-ip-60-cluster/cis-deployment)

## F5 IPAM Deploy Configuration Options

### Step 2

* --orchestration=kubernetes

The orchestration parameter holds the orchestration environment i.e. Kubernetes

* --ip-range="10.192.75.111/24-10.192.75.115/24"

ip-range parameter holds the IP address ranges and from this range, it creates a pool of IP address range which gets allocated to the corresponding hostname in the virtual server CRD.

* --log-level=debug

```
- args:
    - --orchestration=kubernetes
    - --ip-range="10.192.75.111/24-10.192.75.115/24"
    - --log-level=DEBUG
```

Deploy RBAC, schema and F5 IPAM Controller deployment

```
kubectl create -f f5-ipam-rbac.yaml
kubectl create -f f5-ipam-schema.yaml
kubectl create -f f5-ipam-deployment.yaml
```
## Logging output when deploying the F5 IPAM Controller

```
2021/02/12 17:52:49 [DEBUG] Creating IPAM Kubernetes Client
2021/02/12 17:52:49 [DEBUG] [ipam] Creating Informers for Namespace kube-system
2021/02/12 17:52:49 [DEBUG] Created New IPAM Client
2021/02/12 17:52:49 [DEBUG] [MGR] Creating Manager with Provider: f5-ip-provider
2021/02/12 17:52:49 [DEBUG] [PROV] Parsing IP Ranges: 10.192.75.111/24-10.192.75.115/24
2021/02/12 17:52:49 [DEBUG] [PROV] IP Pool: 10.192.75.111 to 10.192.75.115/24
2021/02/12 17:52:49 [DEBUG] [PROV] Processed CIDR: 10.192.75.0/24
2021/02/12 17:52:49 [DEBUG] [STORE] Column names: [id ipaddress status cidr]
2021/02/12 17:52:49 [DEBUG] [STORE] ipaddress_range: 1   10.192.75.111  1       10.192.75.0/24
2021/02/12 17:52:49 [DEBUG] [STORE] ipaddress_range: 2   10.192.75.112  1       10.192.75.0/24
2021/02/12 17:52:49 [DEBUG] [STORE] ipaddress_range: 3   10.192.75.113  1       10.192.75.0/24
2021/02/12 17:52:49 [DEBUG] [STORE] ipaddress_range: 4   10.192.75.114  1       10.192.75.0/24
2021/02/12 17:52:49 [DEBUG] [STORE] ipaddress_range: 5   10.192.75.115  1       10.192.75.0/24
I0212 17:52:49.103363       1 shared_informer.go:197] Waiting for caches to sync for F5 IPAMClient Controller
2021/02/12 17:52:49 [INFO] [CORE] Controller started
2021/02/12 17:52:49 [INFO] Starting IPAMClient Informer
2021/02/12 17:52:49 [DEBUG] Enqueueing on Create: kube-system/ipam.k8s
I0212 17:52:49.211011       1 shared_informer.go:204] Caches are synced for F5 IPAMClient Controller
2021/02/12 17:52:49 [DEBUG] K8S Orchestrator Started
2021/02/12 17:52:49 [DEBUG] Starting Custom Resource Worker
2021/02/12 17:52:49 [DEBUG] Processing Key: &{0xc0004ab080 <nil> Create}
2021/02/12 17:52:49 [DEBUG] Starting Response Worker
```

ipam-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.3/ipam/crd/big-ip-60-cluster/ipam-deployment)


## Configuring CIS CRD to work with F5 IPAM Controller for the following hosts

- hostname "mysite.f5demo.com"
- hostname "myapp.f5demo.com"

### Step 3

Provide a label CIDR: "10.192.75.0/24" in the virtual server CRD. Make your to install the latest CIS virtualserver schema

```
apiVersion: "cis.f5.com/v1"
kind: VirtualServer
metadata:
 name: f5-demo
 labels:
   f5cr: "true"
spec:
 host: mysite.f5demo.com
 cidr: "10.192.75.0/24"
 pools:
 - path: /
   service: f5-demo
   servicePort: 80

```

and

```
apiVersion: "cis.f5.com/v1"
kind: VirtualServer
metadata:
 name: f5-demo
 labels:
   f5cr: "true"
spec:
 host: myapp.f5demo.com
 cidr: "10.192.75.0/24"
 pools:
 - path: /
   service: f5-demo
   servicePort: 80

```

Deploy the CRD and updated schema

```
kubectl create -f customresourcedefinitions.yaml
kubectl create -f vs-mysite.yaml
kubectl create -f vs-myapp.yaml
```

crd-example [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.3/ipam/crd/big-ip-60-cluster/crd-example)

## Logging output when the virtualserver is created

```
2021/02/08 21:57:50 [INFO] [STORE] No A record with Host: mysite.f5demo.com
2021/02/08 21:57:50 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/02/08 21:57:50 [DEBUG] Processing Key: &{0xc0004d8580 0xc0004d8420 Update}
2021/02/08 21:57:50 [DEBUG] [CORE] Allocated IP: 10.192.75.111 for CIDR: 10.192.75.0/24
2021/02/08 21:57:50 [DEBUG] [PROV] Created 'A' Record. Host:mysite.f5demo.com, IP:10.192.75.111
2021/02/08 21:57:50 [DEBUG] Updated: kube-system/ipam.k8s with Status. Added Host: mysite.f5demo.com, CIDR: 10.192.75.0/24, IP: 10.192.75.111
2021/02/08 21:57:50 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/02/08 21:57:50 [DEBUG] Processing Key: &{0xc0004d9b80 0xc0004d8580 Update}
2021/02/08 21:59:55 [INFO] [STORE] No A record with Host: myapp.f5demo.com
2021/02/08 21:59:55 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/02/08 21:59:55 [DEBUG] Processing Key: &{0xc0004d8840 0xc0004d9b80 Update}
2021/02/08 21:59:55 [DEBUG] [CORE] Allocated IP: 10.192.75.112 for CIDR: 10.192.75.0/24
2021/02/08 21:59:55 [DEBUG] [PROV] Created 'A' Record. Host:myapp.f5demo.com, IP:10.192.75.112
2021/02/08 21:59:55 [DEBUG] Updated: kube-system/ipam.k8s with Status. Added Host: myapp.f5demo.com, CIDR: 10.192.75.0/24, IP: 10.192.75.112
2021/02/08 21:59:55 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/02/08 21:59:55 [DEBUG] Processing Key: &{0xc0004349a0 0xc0004d8840 Update}
```

## View the F5 IPAM Controller configuration

F5 IPAM Controller creates the following CRD to create the configuration between CIS and IPAM 

```
[kube@k8s-1-19-master crd-example]$ kubectl describe f5ipam -n kube-system
Name:         ipam.k8s
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>
API Version:  fic.f5.com/v1
Kind:         F5IPAM
Metadata:
  Creation Timestamp:  2021-02-12T17:38:54Z
  Generation:          13
  Managed Fields:
    API Version:  fic.f5.com/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:hostSpecs:
      f:status:
        .:
        f:IPStatus:
    Manager:         Go-http-client
    Operation:       Update
    Time:            2021-02-12T19:05:56Z
  Resource Version:  35195953
  Self Link:         /apis/fic.f5.com/v1/namespaces/kube-system/f5ipams/ipam.k8s
  UID:               fb932c42-aea5-48ed-bd95-c12f58a2cd29
Spec:
  Host Specs:
    Cidr:  10.192.75.0/24
    Host:  myapp.f5demo.com
    Cidr:  10.192.75.0/24
    Host:  mysite.f5demo.com
Status:
  IP Status:
    Cidr:  10.192.75.0/24
    Host:  myapp.f5demo.com
    Ip:    10.192.75.112
    Cidr:  10.192.75.0/24
    Host:  mysite.f5demo.com
    Ip:    10.192.75.111
Events:    <none>
```

View the F5 IPAM CRD and allocate IP status

```
kubectl describe f5ipam -n kube-system
```

## Limitations

CIS cannot update and delete the hostname in the F5-IPAM custom resource hence update and deletion of IP address for virtual server custom may not work as expected. In case if the user wants to reflect the changes, the user can delete the F5-IPAM custom resource from kube-system named "f5ipam" and restart both the controllers

Locate and delete the f5ipam crd before restarting the F5 IPAM Controller 

```
[kube@k8s-1-19-master crd-example]$ kubectl get f5ipam -n kube-system
NAME AGE
ipam.k8s 66m

[kube@k8s-1-19-master crd-example]$ kubectl delete f5ipam ipam.k8s -n kube-system
f5ipam.fic.f5.com "ipam.k8s" deleted
```