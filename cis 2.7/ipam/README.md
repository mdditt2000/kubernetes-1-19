# CIS/IPAM Learning Session

The purpose of this document is to share the content for the CIS/IPAM learning session

Presentation [ppt](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/ipam/documents/CIS%20iPAM%20Learning%20Session.pptx)

## Product Documentation

* [F5 IPAM Controller](https://clouddocs.f5.com/containers/latest/userguide/ipam/)
* [F5 IPAM GitHub](https://github.com/F5Networks/f5-ipam-controller)

## User-guides and Demos

* [F5 IPAM Controller and F5 Container Ingress Services (CIS) using Infoblox IPAM Integration](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/ipam-infoblox/README.md)
* [F5 IPAM Controller User Guide](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/ipam/README.md)
* [Service Type LoadBalancer](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/servicetypelb/README.md)
* [Simplifying Kubernetes Ingress using F5 Technologies](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/simplifying-ingress#readme)

## IPAM Troubleshooting

For IPAM and CIS to work together correctly both controller needs the correct RBAC ServiceAccounts and Bindings. Please make sure you have the correct RBAC created as shown below

* [CIS RBAC](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/ipam/demo/cis/cis-deployment/bigip-ctlr-clusterrole.yaml)
* [IPAM RBAC](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/ipam/demo/ipam-deployment/f5-ipam-rbac.yaml)

### Step 1

Validate **PersistentVolume** 

- Check the name for both the **PersistentVolume** and **PersistentVolumeClaim**
- Validate that the names for **PersistentVolume** and **PersistentVolumeClaim** match correctly

```
❯ kubectl get pv -n kube-system
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS    REASON   AGE
local-pv   1Gi        RWO            Delete           Bound    kube-system/pvc-local   local-storage            12h
❯ kubectl get pvc -n kube-system
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
pvc-local   Bound    local-pv   1Gi        RWO            local-storage   12h
```

- Validate the node where the **PersistentVolume** is created. In my example the node is **k8s-1-19-node1.example.com** as shown below

```
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-1-19-node1.example.com
```
- Validate the path in the **PersistentVolume** where IPAM will store data. In my example the path is **/tmp/cis_ipam** on **k8s-1-19-node1.example.com** as shown below

```
  storageClassName: local-storage
  local:
    path: /tmp/cis_ipam
```
- Validate the temp and file

```
[root@k8s-1-19-node1 cis_ipam]# pwd
/tmp/cis_ipam
[root@k8s-1-19-node1 cis_ipam]# ls
cis_ipam.sqlite3
```
**Note** if [f5-ipam-persitentvolume.yaml](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/ipam/demo/ipam-deployment/f5-ipam-persitentvolume.yaml) is removed the folder gets removed. The temp folder will need to be recreated

### Step 2

Validate that CIS and IPAM controllers are running. You the following kubectl command

```
❯ kubectl get pods --all-namespaces -owide |grep -i 'cis\|ipam\|dep'
kube-system   f5-ipam-controller-5579cdb756-bs6lt                        1/1     Running   0          9h      10.244.1.251     k8s-1-19-node1.example.com    <none>           <none>
kube-system   k8s-bigip-ctlr-deployment-7cb9d788bf-s7hrg                 1/1     Running   0          9h      10.244.1.250     k8s-1-19-node1.example.com    <none>           <none>
```

### Step 3

Validate that CIS and IPAM controllers are using the correct clusterroles. You the following kubectl command. **Note** that CIS is using **bigip-ctlr** ServiceAccount while IPAM is is **ipam-ctlr**

```
❯ kubectl describe clusterrole bigip-ctlr -n kube-system | grep ipam
  ipams.fic.f5.com/status                         []                 []              [get list watch update create patch delete]
  ipams.fic.f5.com                                []                 []              [get list watch update create patch delete]

❯ kubectl describe clusterrole ipam-ctlr -n kube-system | grep ipam
Name:         ipam-ctlr-clusterrole
  ipams.fic.f5.com/status  []                 []              [get list watch update patch]
  ipams.fic.f5.com         []                 []              [get list watch update patch]
```

### Step 4 

Validate the log output when creating the CRD. Use the following log command below

```
❯ kubectl logs -f deploy/k8s-bigip-ctlr-deployment -n kube-system | grep --color=auto -i '\[as3\|posting\|response from\|ipam'
2021/12/17 05:38:52 [DEBUG] [AS3] No certs appended, using only system certs
2021/12/17 05:38:55 [INFO] Posting GET BIGIP AS3 Version request on https://192.168.200.60/mgmt/shared/appsvcs/info
2021/12/17 05:38:55 [DEBUG] [ipam] Created IPAM Custom Resource:
&{{ } {k8s-bigip-ctlr-deployment.k8s.ipam  kube-system /apis/fic.f5.com/v1/namespaces/kube-system/ipams/k8s-bigip-ctlr-deployment.k8s.ipam c5aa891b-1a61-47d5-be3a-bbe573b3f631 104061339 1 2021-12-17 05:40:04 +0000 UTC <nil> <nil> map[] map[] [] []  [{k8s-bigip-ctlr.real Update fic.f5.com/v1 2021-12-17 05:40:04 +0000 UTC FieldsV1 {"f:spec":{},"f:status":{}}}]} {[]} {[]}}
2021/12/17 05:38:55 [INFO] Enqueueing IPAM: &{{IPAM fic.f5.com/v1} {k8s-bigip-ctlr-deployment.k8s.ipam  kube-system /apis/fic.f5.com/v1/namespaces/kube-system/ipams/k8s-bigip-ctlr-deployment.k8s.ipam c5aa891b-1a61-47d5-be3a-bbe573b3f631 104061339 1 2021-12-17 05:40:04 +0000 UTC <nil> <nil> map[] map[] [] []  [{k8s-bigip-ctlr.real Update fic.f5.com/v1 2021-12-17 05:40:04 +0000 UTC FieldsV1 {"f:spec":{},"f:status":{}}}]} {[]} {[]}}
I1217 05:38:55.645395       1 shared_informer.go:240] Waiting for caches to sync for F5 IPAMClient Controller
2021/12/17 05:38:55 [DEBUG] Processing Key: &{kube-system IPAM k8s-bigip-ctlr-deployment.k8s.ipam 0xc00081a6e0 false}
2021/12/17 05:38:55 [DEBUG] [ipam] Syncing IPAM dependent virtual servers
2021/12/17 05:38:55 [DEBUG] [ipam] Syncing IPAM dependent transport servers
I1217 05:38:55.745611       1 shared_informer.go:247] Caches are synced for F5 IPAMClient Controller
2021/12/17 05:56:42 [DEBUG] Enqueueing VirtualServer: &{{ } {f5-demo-mysite  default /apis/cis.f5.com/v1/namespaces/default/virtualservers/f5-demo-mysite 32dddf1e-3326-41a8-b176-79596fa1e7a9 104064151 1 2021-12-17 05:57:51 +0000 UTC <nil> <nil> map[f5cr:true] map[] [] []  [{kubectl-create Update cis.f5.com/v1 2021-12-17 05:57:51 +0000 UTC FieldsV1 {"f:metadata":{"f:labels":{".":{},"f:f5cr":{}}},"f:spec":{".":{},"f:host":{},"f:ipamLabel":{},"f:pools":{}}}}]} {mysite.f5demo.com   Test  0 0 [{/ f5-demo 80  {http /  10 31} }]      [] [] [] } { }}
2021/12/17 05:56:42 [DEBUG] [ipam] Updated IPAM CR.
2021/12/17 05:56:42 [DEBUG] Finished syncing virtual servers &{TypeMeta:{Kind: APIVersion:} ObjectMeta:{Name:f5-demo-mysite GenerateName: Namespace:default SelfLink:/apis/cis.f5.com/v1/namespaces/default/virtualservers/f5-demo-mysite UID:32dddf1e-3326-41a8-b176-79596fa1e7a9 ResourceVersion:104064151 Generation:1 CreationTimestamp:2021-12-17 05:57:51 +0000 UTC DeletionTimestamp:<nil> DeletionGracePeriodSeconds:<nil> Labels:map[f5cr:true] Annotations:map[] OwnerReferences:[] Finalizers:[] ClusterName: ManagedFields:[{Manager:kubectl-create Operation:Update APIVersion:cis.f5.com/v1 Time:2021-12-17 05:57:51 +0000 UTC FieldsType:FieldsV1 FieldsV1:{"f:metadata":{"f:labels":{".":{},"f:f5cr":{}}},"f:spec":{".":{},"f:host":{},"f:ipamLabel":{},"f:pools":{}}}}]} Spec:{Host:mysite.f5demo.com HostGroup: VirtualServerAddress: IPAMLabel:Test VirtualServerName: VirtualServerHTTPPort:0 VirtualServerHTTPSPort:0 Pools:[{Path:/ Service:f5-demo ServicePort:80 NodeMemberLabel: Monitor:{Type:http Send:/ Recv: Interval:10 Timeout:31} Rewrite:}] TLSProfileName: HTTPTraffic: SNAT: WAF: RewriteAppRoot: AllowVLANs:[] IRules:[] ServiceIPAddress:[] PolicyName:} Status:{VSAddress: StatusOk:}} (10.129434ms)
2021/12/17 05:56:42 [DEBUG] Processing Key: &{kube-system IPAM k8s-bigip-ctlr-deployment.k8s.ipam 0xc00081ac60 false}
2021/12/17 05:56:42 [INFO] Enqueueing Updated IPAM: &{{ } {k8s-bigip-ctlr-deployment.k8s.ipam  kube-system /apis/fic.f5.com/v1/namespaces/kube-system/ipams/k8s-bigip-ctlr-deployment.k8s.ipam c5aa891b-1a61-47d5-be3a-bbe573b3f631 104064153 2 2021-12-17 05:40:04 +0000 UTC <nil> <nil> map[] map[] [] []  [{f5-ipam-controller Update fic.f5.com/v1 2021-12-17 05:57:51 +0000 UTC FieldsV1 {"f:status":{".":{},"f:IPStatus":{}}}} {k8s-bigip-ctlr.real Update fic.f5.com/v1 2021-12-17 05:57:51 +0000 UTC FieldsV1 {"f:spec":{".":{},"f:hostSpecs":{}}}}]} {[0xc000783650]} {[0xc000b72780]}}
2021/12/17 05:56:42 [DEBUG] [ipam] Syncing IPAM dependent virtual servers
2021/12/17 05:56:42 [DEBUG] Finished syncing virtual servers &{TypeMeta:{Kind: APIVersion:} ObjectMeta:{Name:f5-demo-mysite GenerateName: Namespace:default SelfLink:/apis/cis.f5.com/v1/namespaces/default/virtualservers/f5-demo-mysite UID:32dddf1e-3326-41a8-b176-79596fa1e7a9 ResourceVersion:104064151 Generation:1 CreationTimestamp:2021-12-17 05:57:51 +0000 UTC DeletionTimestamp:<nil> DeletionGracePeriodSeconds:<nil> Labels:map[f5cr:true] Annotations:map[] OwnerReferences:[] Finalizers:[] ClusterName: ManagedFields:[{Manager:kubectl-create Operation:Update APIVersion:cis.f5.com/v1 Time:2021-12-17 05:57:51 +0000 UTC FieldsType:FieldsV1 FieldsV1:{"f:metadata":{"f:labels":{".":{},"f:f5cr":{}}},"f:spec":{".":{},"f:host":{},"f:ipamLabel":{},"f:pools":{}}}}]} Spec:{Host:mysite.f5demo.com HostGroup: VirtualServerAddress: IPAMLabel:Test VirtualServerName: VirtualServerHTTPPort:0 VirtualServerHTTPSPort:0 Pools:[{Path:/ Service:f5-demo ServicePort:80 NodeMemberLabel: Monitor:{Type:http Send:/ Recv: Interval:10 Timeout:31} Rewrite:}] TLSProfileName: HTTPTraffic: SNAT: WAF: RewriteAppRoot: AllowVLANs:[] IRules:[] ServiceIPAddress:[] PolicyName:} Status:{VSAddress:10.192.75.117 StatusOk:}} (4.451673ms)
2021/12/17 05:56:42 [DEBUG] [ipam] Syncing IPAM dependent transport servers
2021/12/17 05:56:42 [DEBUG] [AS3] PostManager Accepted the configuration
2021/12/17 05:56:42 [DEBUG] [AS3] posting request to https://192.168.200.60/mgmt/shared/appsvcs/declare/
2021/12/17 05:56:47 [DEBUG] [AS3] Response from BIG-IP: code: 200 --- tenant:k8s --- message: success
2021/12/17 05:56:47 [DEBUG] Enqueueing VirtualServer: &{{ } {f5-demo-mysite  default /apis/cis.f5.com/v1/namespaces/default/virtualservers/f5-demo-mysite 32dddf1e-3326-41a8-b176-79596fa1e7a9 104064166 1 2021-12-17 05:57:51 +0000 UTC <nil> <nil> map[f5cr:true] map[] [] []  [{kubectl-create Update cis.f5.com/v1 2021-12-17 05:57:51 +0000 UTC FieldsV1 {"f:metadata":{"f:labels":{".":{},"f:f5cr":{}}},"f:spec":{".":{},"f:host":{},"f:ipamLabel":{},"f:pools":{}}}} {k8s-bigip-ctlr.real Update cis.f5.com/v1 2021-12-17 05:57:56 +0000 UTC FieldsV1 {"f:status":{".":{},"f:status":{},"f:vsAddress":{}}}}]} {mysite.f5demo.com   Test  0 0 [{/ f5-demo 80  {http /  10 31} }]      [] [] [] } {10.192.75.117 Ok}}
2021/12/17 05:56:47 [DEBUG] Finished syncing virtual servers &{TypeMeta:{Kind: APIVersion:} ObjectMeta:{Name:f5-demo-mysite GenerateName: Namespace:default SelfLink:/apis/cis.f5.com/v1/namespaces/default/virtualservers/f5-demo-mysite UID:32dddf1e-3326-41a8-b176-79596fa1e7a9 ResourceVersion:104064166 Generation:1 CreationTimestamp:2021-12-17 05:57:51 +0000 UTC DeletionTimestamp:<nil> DeletionGracePeriodSeconds:<nil> Labels:map[f5cr:true] Annotations:map[] OwnerReferences:[] Finalizers:[] ClusterName: ManagedFields:[{Manager:kubectl-create Operation:Update APIVersion:cis.f5.com/v1 Time:2021-12-17 05:57:51 +0000 UTC FieldsType:FieldsV1 FieldsV1:{"f:metadata":{"f:labels":{".":{},"f:f5cr":{}}},"f:spec":{".":{},"f:host":{},"f:ipamLabel":{},"f:pools":{}}}} {Manager:k8s-bigip-ctlr.real Operation:Update APIVersion:cis.f5.com/v1 Time:2021-12-17 05:57:56 +0000 UTC FieldsType:FieldsV1 FieldsV1:{"f:status":{".":{},"f:status":{},"f:vsAddress":{}}}}]} Spec:{Host:mysite.f5demo.com HostGroup: VirtualServerAddress: IPAMLabel:Test VirtualServerName: VirtualServerHTTPPort:0 VirtualServerHTTPSPort:0 Pools:[{Path:/ Service:f5-demo ServicePort:80 NodeMemberLabel: Monitor:{Type:http Send:/ Recv: Interval:10 Timeout:31} Rewrite:}] TLSProfileName: HTTPTraffic: SNAT: WAF: RewriteAppRoot: AllowVLANs:[] IRules:[] ServiceIPAddress:[] PolicyName:} Status:{VSAddress:10.192.75.117 StatusOk:Ok}} (4.463156ms)
2021/12/17 05:56:47 [DEBUG] [AS3] No Change in the Configuration
```
### Step 5 

View the IPAM CRD. Validate the IPStatus 

```
❯ kubectl get ipams -n kube-system  -oyaml
apiVersion: v1
items:
- apiVersion: fic.f5.com/v1
  kind: IPAM
  metadata:
    creationTimestamp: "2021-12-17T05:40:04Z"
    generation: 2
    managedFields:
    - apiVersion: fic.f5.com/v1
      fieldsType: FieldsV1
      fieldsV1:
        f:status:
          .: {}
          f:IPStatus: {}
      manager: f5-ipam-controller
      operation: Update
      time: "2021-12-17T05:57:51Z"
    - apiVersion: fic.f5.com/v1
      fieldsType: FieldsV1
      fieldsV1:
        f:spec:
          .: {}
          f:hostSpecs: {}
      manager: k8s-bigip-ctlr.real
      operation: Update
      time: "2021-12-17T05:57:51Z"
    name: k8s-bigip-ctlr-deployment.k8s.ipam
    namespace: kube-system
    resourceVersion: "104064153"
    selfLink: /apis/fic.f5.com/v1/namespaces/kube-system/ipams/k8s-bigip-ctlr-deployment.k8s.ipam
    uid: c5aa891b-1a61-47d5-be3a-bbe573b3f631
  spec:
    hostSpecs:
    - host: mysite.f5demo.com
      ipamLabel: Test
  status:
    IPStatus:
    - host: mysite.f5demo.com
      ip: 10.192.75.117
      ipamLabel: Test
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

### Step 6 

Seeing the following error

```
❯ kubectl logs -f deploy/f5-ipam-controller -n kube-system
2022/01/06 00:16:12 [DEBUG] Creating IPAM Kubernetes Client
2022/01/06 00:16:12 [INFO] [INIT] Starting: F5 IPAM Controller - Version: 0.1.6, BuildInfo: azure-1677-f86d2913adf51b4c8ebc04cac919203623abe5d6
2022/01/06 00:16:12 [DEBUG] [ipam] Creating Informers for Namespace kube-system
2022/01/06 00:16:12 [DEBUG] Created New IPAM Client
2022/01/06 00:16:12 [DEBUG] [MGR] Creating Manager with Provider: f5-ip-provider
2022/01/06 00:16:12 [DEBUG] [STORE] Using IPAM DB file from mount path
2022/01/06 00:16:12 [DEBUG] [STORE] [ipaddress status ipam_label reference]
2022/01/06 00:16:12 [DEBUG] [STORE] 10.192.75.117 0 Test mysite.f5demo.com
2022/01/06 00:16:12 [DEBUG] [STORE] 10.192.75.118 1 Test lWbHTRADfE17uwQH
2022/01/06 00:16:12 [DEBUG] [STORE] 10.192.75.119 1 Test gYVa2GgdDYbR6R4A
2022/01/06 00:16:12 [DEBUG] [PROV] Provider Initialised
I0106 00:16:12.548751       1 shared_informer.go:240] Waiting for caches to sync for F5 IPAMClient Controller
2022/01/06 00:16:12 [INFO] [CORE] Controller started
2022/01/06 00:16:12 [INFO] Starting IPAMClient Informer
2022/01/06 00:16:12 [DEBUG] Enqueueing on Create: kube-system/k8s-bigip-ctlr-deployment.k8s.ipam
I0106 00:16:12.649004       1 shared_informer.go:247] Caches are synced for F5 IPAMClient Controller
2022/01/06 00:16:12 [DEBUG] K8S Orchestrator Started
2022/01/06 00:16:12 [DEBUG] Starting Custom Resource Worker
2022/01/06 00:16:12 [DEBUG] Processing Key: &{0xc0001dc160 <nil> Create}
2022/01/06 00:16:12 [DEBUG] Starting Response Worker
2022/01/06 00:16:12 [ERROR] Unable to Update IPAM: kube-system/k8s-bigip-ctlr-deployment.k8s.ipam        Error: ipams.fic.f5.com "k8s-bigip-ctlr-deployment.k8s.ipam" not found
2022/01/06 00:16:12 [DEBUG] Updated: kube-system/k8s-bigip-ctlr-deployment.k8s.ipam with Status. With IP: 10.192.75.117 for Request:
Hostname: mysite.f5demo.com     Key:    IPAMLabel: Test IPAddr:         Operation: Create
```

- Remove the IPAM schema required in CIS 2.7
- Restart CIS and IPAM