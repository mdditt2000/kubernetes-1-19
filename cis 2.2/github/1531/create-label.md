## No label created

```
[kube@k8s-1-19-master crd-examples]$ kubectl get nodes
NAME                               STATUS   ROLES    AGE   VERSION
k8s-1-19-master.example.com   Ready    master   29d   v1.19.0
k8s-1-19-node1.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node2.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node3.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node4.example.com    Ready    <none>   29d   v1.19.0
```
Create the label
```
kubectl label nodes k8s-1-19-node1.example.com f5role=worker
ubectl label nodes k8s-1-19-node2.example.com f5role=worker
kubectl label nodes k8s-1-19-node3.example.com f5role=worker
kubectl label nodes k8s-1-19-node4.example.com f5role=worker
```

Below you can see the label created **f5role=worker** for nodes 1 - 4

```
[kube@k8s-1-19-master 1531]$ kubectl get nodes --show-labels
NAME                               STATUS   ROLES    AGE   VERSION   LABELS
k8s-1-19-master.example.com   Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-master.example.com,kubernetes.io/os=linux
k8s-1-19-node1.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node1.example.com,kubernetes.io/os=linux
k8s-1-19-node2.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node2.example.com,kubernetes.io/os=linux
k8s-1-19-node3.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node3.example.com,kubernetes.io/os=linux
k8s-1-19-node4.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node4.example.com,kubernetes.io/os=linux
```








```
