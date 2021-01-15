# F5 IPAM Controller for CIS 2.2.2

The F5 IPAM Controller is a Docker container that allocates IP addresses from an static list for hostnames. The F5 IPAM Controller watches CRD resources and consumes the hostnames within each resource.

The Controller can:

* Allocate IP address from static IP address pool based on the CIDR mentioned in a Kubernetes resource

* F5 IPAM Controller decides to allocate the IP from the respective IP address pool for the hostname specified in the virtualserver custom resource

**Note** The idea here is that we will support CRD, Type LB and possibly  route/ingress in the future

## F5 CIS Configuration Options for IPAM Deployment

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

Files located at https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.2.2/ipam/crd/big-ip-60-cluster/cis-deployment

## F5 IPAM Deploy Configuration Options

### Step 2

* --orchestration=kubernetes

The orchestration parameter holds the orchestration environment i.e. Kubernetes.

* --ip-range="10.192.75.111/24-10.192.75.115/24"28"

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

Files located at https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.2.2/ipam/crd/big-ip-60-cluster/ipam-deployment

## Configuring CIS CRD to work with F5 IPAM Controller

### Step 3

Provide a parameter CIDR: "10.192.75.0/24" in the virtual server CRD. Make your to install the latest CIS virtualserver schema

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
Deploy the CRD and updated schema

```
kubectl create -f customresourcedefinitions.yaml
kubectl create -f virtual-server-crd.yaml
kubec

Files located at https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.2.2/ipam/crd/big-ip-60-cluster/crd-example

## Logging output when creating the CRD

```
[kube@k8s-1-19-master root]$ kubectl logs -f deploy/f5-ipam-controller -n kube-system
2021/01/14 19:09:29 [DEBUG] Creating IPAM Kubernetes Client
2021/01/14 19:09:29 [DEBUG] [ipam] Creating Informers for Namespace kube-system
2021/01/14 19:09:29 [DEBUG] Created New IPAM Client
2021/01/14 19:09:29 [DEBUG] [MGR] Creating Manager with Provider: f5-ip-provider
2021/01/14 19:09:29 [DEBUG] [PROV] Parsing IP Ranges: 10.192.75.111/24-10.192.75.115/24
2021/01/14 19:09:29 [DEBUG] [PROV] IP Pool: 10.192.75.111 to 10.192.75.115/24
2021/01/14 19:09:29 [DEBUG] [PROV] Processed CIDR: 10.192.75.0/24
2021/01/14 19:09:29 [INFO] [CORE] Controller started
2021/01/14 19:09:29 [DEBUG] [STORE] Column names: [id ipaddress status cidr]
2021/01/14 19:09:29 [DEBUG] [STORE] ipaddress_range: 1   10.192.75.111  1       10.192.75.0/24
2021/01/14 19:09:29 [DEBUG] [STORE] ipaddress_range: 2   10.192.75.112  1       10.192.75.0/24
2021/01/14 19:09:29 [DEBUG] [STORE] ipaddress_range: 3   10.192.75.113  1       10.192.75.0/24
2021/01/14 19:09:29 [DEBUG] [STORE] ipaddress_range: 4   10.192.75.114  1       10.192.75.0/24
2021/01/14 19:09:29 [DEBUG] [STORE] ipaddress_range: 5   10.192.75.115  1       10.192.75.0/24
2021/01/14 19:09:29 [INFO] Starting IPAMClient Informer
I0114 19:09:29.166284       1 shared_informer.go:197] Waiting for caches to sync for F5 IPAMClient Controller
I0114 19:09:29.270816       1 shared_informer.go:204] Caches are synced for F5 IPAMClient Controller
2021/01/14 19:09:29 [DEBUG] K8S Orchestrator Started
2021/01/14 19:09:29 [DEBUG] Starting Custom Resource Worker
2021/01/14 19:09:29 [DEBUG] Starting Response Worker
2021/01/14 21:14:30 [DEBUG] Enqueueing on Create: kube-system/ipam.k8s
2021/01/14 21:14:30 [DEBUG] Processing Key: &{0xc00032e000 <nil> Create}
2021/01/14 21:14:30 [INFO] [STORE] No A record with Host: mysite.f5demo.com
2021/01/14 21:14:30 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/01/14 21:14:30 [DEBUG] Processing Key: &{0xc00032e2c0 0xc00032e000 Update}
2021/01/14 21:14:30 [DEBUG] [CORE] Allocated IP: 10.192.75.111 for CIDR: 10.192.75.0/24
2021/01/14 21:14:30 [DEBUG] [PROV] Created 'A' Record. Host:mysite.f5demo.com, IP:10.192.75.111
2021/01/14 21:14:30 [DEBUG] Updated: kube-system/ipam.k8s with Status. Added Host: mysite.f5demo.com, CIDR: 10.192.75.0/24, IP: 10.192.75.111
2021/01/14 21:14:30 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/01/14 21:14:30 [DEBUG] Processing Key: &{0xc00032f340 0xc00032e2c0 Update}
2021/01/14 21:16:21 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/01/14 21:16:21 [DEBUG] Processing Key: &{0xc00032e000 0xc00032f340 Update}
2021/01/14 21:16:21 [DEBUG] Updated: kube-system/ipam.k8s with Status. Added Host: mysite.f5demo.com, CIDR: , IP: 10.192.75.111
2021/01/14 21:16:21 [DEBUG] Enqueueing on Update: kube-system/ipam.k8s
2021/01/14 21:16:21 [DEBUG] Processing Key: &{0xc0001ae000 0xc00032e000 Update}
```
