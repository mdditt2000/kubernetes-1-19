# Service Type LB with PolicyCRD

A service of type LoadBalancer is the simplest and the fastest way to expose a service inside a Kubernetes cluster to the external world. All you need to-do is specify the service type as type=LoadBalancer in the service definition. 

Services of type LoadBalancer are natively supported in Kubernetes deployments. When you create a service of type LoadBalancer, Kubernetes spins up a service in integration with F5 IPAM Controller which allocates an IP address from the ip-range specified by the **ipamlabel**. Using CIS with services configured for type LoadBalancer, BIG-IP can load balance the incoming traffic to the Kubernetes cluster without having to create any ingress resource. CIS will manage the public IP addresses for the application using the F5 IPAM Controller. This cloud like simplification of load balancer resources could significantly reduce your operational expenses. 

With simplicity comes reduced completely or features. Services of type LoadBalancer lacks the capability of advanced configuration. However CIS can solve this with adding a **PolicyCRD** to the Virtual Address that is created on BIG-IP. Ths advanced configuration could be adding **TCP profiles, iRule, SNAT pools, logging, firewall and DOS profiles etc**. This examples demonstrates adding a firewall policy to protect the Kubernetes cluster from Network attacks. Logging policy is also added to log any attacks

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/advanced_servicetypelb/diagram/2022-05-15_13-23-45.png)

Demo on YouTube [video]()

Looking at the diagram and Service of type LoadBalancer, the following events occur:

1. CIS will update the service whenever the loadBalancer IP in the service is empty.
2. The IPAM controller assigns an IP address for the loadBalancer: ingress: object from the ip-range based on the ipamlabel specified but the annotation
3. Once the object is updated with the IP address, CIS automatically configures BIG-IP with the External IP address as shown below

#### Example of Service type LoadBalancer + Policy CRD shown in the diagram

```
apiVersion: v1
items:
- apiVersion: fic.f5.com/v1
  kind: IPAM
  metadata:
    creationTimestamp: "2022-05-13T05:52:12Z"
    generation: 14
    managedFields:
    - apiVersion: fic.f5.com/v1
      fieldsType: FieldsV1
      fieldsV1:
        f:status:
          .: {}
          f:IPStatus: {}
      manager: f5-ipam-controller
      operation: Update
      time: "2022-05-15T19:47:07Z"
    - apiVersion: fic.f5.com/v1
      fieldsType: FieldsV1
      fieldsV1:
        f:spec:
          .: {}
          f:hostSpecs: {}
      manager: k8s-bigip-ctlr
      operation: Update
      time: "2022-05-15T19:47:07Z"
    name: k8s-bigip-ctlr-deployment.k8s.ipam
    namespace: kube-system
    resourceVersion: "138504840"
    selfLink: /apis/fic.f5.com/v1/namespaces/kube-system/ipams/k8s-bigip-ctlr-deployment.k8s.ipam
    uid: 32f09f3e-daa4-4fd8-8f0a-f4700d8c9be2
  spec:
    hostSpecs:
    - ipamLabel: Test
      key: default/f5-demo-test_svc
  status:
    IPStatus:
    - ip: 10.192.75.117
      ipamLabel: Test
      key: default/f5-demo-test_svc
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""

```

## Create the CIS Deployment Configuration

### Step 1

Add the parameter --ipam=true in the CIS deployment to provide the integration with CIS and IPAM

* --ipam=true

```
args: 
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

Deploy CIS and CRD schema

```
kubectl create -f bigip-ctlr-clusterrole.yaml
kubectl create -f f5-bigip-ctlr-deployment.yaml
kubectl create -f customresourcedefinitions.yaml
```

* cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/advanced_servicetypelb/cis-deployment)
* crd-schema [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/advanced_servicetypelb/crd-schema)

## F5 IPAM Deployment Configuration

### Step 2

* --orchestration=kubernetes

The orchestration parameter holds the orchestration environment i.e. Kubernetes

* --ip-range='{"Test":"10.192.75.117-10.192.75.119"}'

ip-range parameter holds the IP address ranges and from this range, it creates a pool of IP address range which gets allocated by the **ipamlabel** defined in the Service

* --log-level=debug

```
- args:
    - --orchestration=kubernetes
    - --ip-range='{"Test":"10.192.75.117-10.192.75.119"}'
    - --log-level=DEBUG
```

Deploy RBAC, schema and F5 IPAM Controller deployment

```
kubectl create -f f5-ipam-rbac.yaml
kubectl create -f f5-ipam-persitentvolume.yaml
kubectl create -f f5-ipam-deployment.yaml
```
## Logging output when deploying the F5 IPAM Controller

```
[kube@k8s-1-19-master crd-example]$ kubectl logs -f deploy/f5-ipam-controller -n kube-system
❯ kubectl logs -f deploy/f5-ipam-controller -n kube-system
2022/05/12 22:29:12 [INFO] [INIT] Starting: F5 IPAM Controller - Version: 0.1.7, BuildInfo: azure-2264-f033b96f43347f6c527b24989fa55f535d0258f9
2022/05/12 22:29:12 [DEBUG] Creating IPAM Kubernetes Client
2022/05/12 22:29:12 [DEBUG] [ipam] Creating Informers for Namespace kube-system
2022/05/12 22:29:12 [DEBUG] Created New IPAM Client
2022/05/12 22:29:12 [DEBUG] [MGR] Creating Manager with Provider: f5-ip-provider
2022/05/12 22:29:12 [DEBUG] [STORE] Using IPAM DB file from mount path
2022/05/12 22:29:12 [INFO] [CORE] Controller started
2022/05/12 22:29:12 [INFO] Starting IPAMClient Informer
2022/05/12 22:29:12 [DEBUG] [STORE] [ipaddress status ipam_label reference]
2022/05/12 22:29:12 [DEBUG] [STORE] 10.192.75.117 0 Test default/f5-demo-test_svc
2022/05/12 22:29:12 [DEBUG] [STORE] 10.192.75.118 1 Test lWbHTRADfE17uwQH
2022/05/12 22:29:12 [DEBUG] [STORE] 10.192.75.119 1 Test gYVa2GgdDYbR6R4A
2022/05/12 22:29:12 [DEBUG] [PROV] Provider Initialised
I0512 22:29:12.055918       1 shared_informer.go:240] Waiting for caches to sync for F5 IPAMClient Controller
2022/05/12 22:29:12 [DEBUG] Enqueueing on Create: kube-system/k8s-bigip-ctlr-deployment.k8s.ipam
I0512 22:29:12.156260       1 shared_informer.go:247] Caches are synced for F5 IPAMClient Controller
2022/05/12 22:29:12 [DEBUG] K8S Orchestrator Started
2022/05/12 22:29:12 [DEBUG] Starting Custom Resource Worker
2022/05/12 22:29:12 [DEBUG] Processing Key: &{0xc0004de6e0 <nil> Create}
2022/05/12 22:29:12 [DEBUG] Starting Response Worker

```

ipam-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/advanced_servicetypelb/ipam-deployment)


## Create the Service Type LoadBalancer Service and add PolicyCRD

### Step 3

Create the pod deployments and services for the test application

```
kubectl create -f f5-demo-test-deployment.yaml
Kubectl create -f policy-type-lb.yaml
kubectl create -f f5-demo-test-service.yaml
```

pod-deployments [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/advanced_servicetypelb/pod-deployment/test)

## Logging output when the IPAM controller when the services are created

```
2022/05/15 19:44:09 [DEBUG] Enqueueing Service: &Service{ObjectMeta:{f5-demo-test  default /api/v1/namespaces/default/services/f5-demo-test df3893d6-0e7f-4fa7-b09e-493391bd93a9 138504836 0 2022-05-15 19:47:07 +0000 UTC <nil> <nil> map[app:f5-demo-test] map[cis.f5.com/health:{"interval": 10, "timeout": 31} cis.f5.com/ipamLabel:Test cis.f5.com/policyName:type-lb] [] []  [{kubectl-create Update v1 2022-05-15 19:47:07 +0000 UTC FieldsV1 {"f:metadata":{"f:annotations":{".":{},"f:cis.f5.com/health":{},"f:cis.f5.com/ipamLabel":{},"f:cis.f5.com/policyName":{}},"f:labels":{".":{},"f:app":{}}},"f:spec":{"f:externalTrafficPolicy":{},"f:ports":{".":{},"k:{\"port\":80,\"protocol\":\"TCP\"}":{".":{},"f:name":{},"f:port":{},"f:protocol":{},"f:targetPort":{}}},"f:selector":{".":{},"f:app":{}},"f:sessionAffinity":{},"f:type":{}}}}]},Spec:ServiceSpec{Ports:[]ServicePort{ServicePort{Name:f5-demo-test,Protocol:TCP,Port:80,TargetPort:{0 80 },NodePort:31340,AppProtocol:nil,},},Selector:map[string]string{app: f5-demo-test,},ClusterIP:10.96.245.18,Type:LoadBalancer,ExternalIPs:[],SessionAffinity:None,LoadBalancerIP:,LoadBalancerSourceRanges:[],ExternalName:,ExternalTrafficPolicy:Cluster,HealthCheckNodePort:0,PublishNotReadyAddresses:false,SessionAffinityConfig:nil,TopologyKeys:[],IPFamilyPolicy:nil,ClusterIPs:[],IPFamilies:[],AllocateLoadBalancerNodePorts:nil,LoadBalancerClass:nil,InternalTrafficPolicy:nil,},Status:ServiceStatus{LoadBalancer:LoadBalancerStatus{Ingress:[]LoadBalancerIngress{},},Conditions:[]Condition{},},}
2022/05/15 19:44:09 [DEBUG] [ipam] Updated IPAM CR.
2022/05/15 19:44:09 [INFO] Enqueueing Updated IPAM: &{{ } {k8s-bigip-ctlr-deployment.k8s.ipam  kube-system /apis/fic.f5.com/v1/namespaces/kube-system/ipams/k8s-bigip-ctlr-deployment.k8s.ipam 32f09f3e-daa4-4fd8-8f0a-f4700d8c9be2 138504840 14 2022-05-13 05:52:12 +0000 UTC <nil> <nil> map[] map[] [] []  [{f5-ipam-controller Update fic.f5.com/v1 2022-05-15 19:47:07 +0000 UTC FieldsV1 {"f:status":{".":{},"f:IPStatus":{}}}} {k8s-bigip-ctlr Update fic.f5.com/v1 2022-05-15 19:47:07 +0000 UTC FieldsV1 {"f:spec":{".":{},"f:hostSpecs":{}}}}]} {[0xc000b307b0]} {[0xc00074a1c0]}}
2022/05/15 19:44:09 [DEBUG] Processing Key: &{kube-system IPAM k8s-bigip-ctlr-deployment.k8s.ipam 0xc000d0f8c0 false}
2022/05/15 19:44:09 [DEBUG] [ipam] Syncing IPAM dependent virtual servers
2022/05/15 19:44:09 [DEBUG] [ipam] Syncing IPAM dependent transport servers
2022/05/15 19:44:09 [DEBUG] Enqueueing Updated Service: &Service{ObjectMeta:{f5-demo-test  default /api/v1/namespaces/default/services/f5-demo-test df3893d6-0e7f-4fa7-b09e-493391bd93a9 138504841 0 2022-05-15 19:47:07 +0000 UTC <nil> <nil> map[app:f5-demo-test] map[cis.f5.com/health:{"interval": 10, "timeout": 31} cis.f5.com/ipamLabel:Test cis.f5.com/policyName:type-lb] [] []  [{k8s-bigip-ctlr Update v1 2022-05-15 19:47:07 +0000 UTC FieldsV1 {"f:status":{"f:loadBalancer":{"f:ingress":{}}}}} {kubectl-create Update v1 2022-05-15 19:47:07 +0000 UTC FieldsV1 {"f:metadata":{"f:annotations":{".":{},"f:cis.f5.com/health":{},"f:cis.f5.com/ipamLabel":{},"f:cis.f5.com/policyName":{}},"f:labels":{".":{},"f:app":{}}},"f:spec":{"f:externalTrafficPolicy":{},"f:ports":{".":{},"k:{\"port\":80,\"protocol\":\"TCP\"}":{".":{},"f:name":{},"f:port":{},"f:protocol":{},"f:targetPort":{}}},"f:selector":{".":{},"f:app":{}},"f:sessionAffinity":{},"f:type":{}}}}]},Spec:ServiceSpec{Ports:[]ServicePort{ServicePort{Name:f5-demo-test,Protocol:TCP,Port:80,TargetPort:{0 80 },NodePort:31340,AppProtocol:nil,},},Selector:map[string]string{app: f5-demo-test,},ClusterIP:10.96.245.18,Type:LoadBalancer,ExternalIPs:[],SessionAffinity:None,LoadBalancerIP:,LoadBalancerSourceRanges:[],ExternalName:,ExternalTrafficPolicy:Cluster,HealthCheckNodePort:0,PublishNotReadyAddresses:false,SessionAffinityConfig:nil,TopologyKeys:[],IPFamilyPolicy:nil,ClusterIPs:[],IPFamilies:[],AllocateLoadBalancerNodePorts:nil,LoadBalancerClass:nil,InternalTrafficPolicy:nil,},Status:ServiceStatus{LoadBalancer:LoadBalancerStatus{Ingress:[]LoadBalancerIngress{LoadBalancerIngress{IP:10.192.75.117,Hostname:,Ports:[]PortStatus{},},},},Conditions:[]Condition{},},}
2022/05/15 19:44:09 [DEBUG] [AS3] PostManager Accepted the configuration
2022/05/15 19:44:09 [DEBUG] [AS3] posting request to https://192.168.200.60/mgmt/shared/appsvcs/declare/
2022/05/15 19:44:24 [DEBUG] [AS3] Response from BIG-IP: code: 200 --- tenant:k8s --- message: success
```

## View the Service Type LoadBalancer status

Use the kubectl get service command to determine the EXTERNAL-IP

```
❯ kubectl get service
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
f5-demo-test   LoadBalancer   10.96.245.18   10.192.75.117   80:31340/TCP   51m
kubernetes     ClusterIP      10.96.0.1      <none>          443/TCP        613d
```
CIS will add the EXTERNAL-IP to the BIG-IP as you can see in the diagram

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/advanced_servicetypelb/diagram/2022-05-15_13-39-58.png)