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


