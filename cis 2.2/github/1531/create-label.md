```
[kube@k8s-1-19-master crd-examples]$ kubectl get nodes
NAME                               STATUS   ROLES    AGE   VERSION
k8s-1-19-master.example.com   Ready    master   29d   v1.19.0
k8s-1-19-node1.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node2.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node3.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node4.example.com    Ready    <none>   29d   v1.19.0

kubectl label nodes k8s-1-19-node1.example.com node-role.kubernetes.io/f5role=worker
kubectl label nodes k8s-1-19-node2.example.com node-role.kubernetes.io/f5role=worker
kubectl label nodes k8s-1-19-node3.example.com node-role.kubernetes.io/f5role=worker
kubectl label nodes k8s-1-19-node4.example.com node-role.kubernetes.io/f5role=worker
```