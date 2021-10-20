## Creating VXLAN Tunnels on Kubernetes Cluster

This configuration is for Standalone BIG-IP

Log in to BIG-IP and create a partition called kubernetes for CIS

    tmsh create auth partition kubernetes

Create a VXLAN profile

    tmsh create net tunnels vxlan fl-vxlan port 8472 flooding-type none

Create a VXLAN tunnel

    tmsh create net tunnels tunnel fl-vxlan key 1 profile fl-vxlan local-address 10.1.1.4

Create the VXLAN tunnel self IP

    tmsh create net self 10.1.1.4 address 10.244.20.4/255.255.0.0 allow-service none vlan fl-vxlan

Save the configuration.

    tmsh save sys config

Before deploying CIS in ClusterIP mode, you need to configure BIG-IP as a node in the Kubernetes cluster. To do so you will need to modify bigip-node.yaml with the MAC address auto-created from the previous steps. From the jumpbox terminal, run the following command at bigip1. Copy the displayed MAC Address.

    tmsh show net tunnels tunnel k8s-tunnel all-properties

Update the MAC address obtained in the previous step to following yaml file:

bigip-node.yaml

```
apiVersion: v1
kind: Node
metadata:
  name: bigip1
  annotations:
    #Replace IP with self IP for your deployment
    flannel.alpha.coreos.com/public-ip: "10.1.1.4"
    #Replace MAC with your BIG-IP Flannel VXLAN Tunnel MAC
    flannel.alpha.coreos.com/backend-data: '{"VtepMAC":"2c:c2:60:23:0c:58"}'
    flannel.alpha.coreos.com/backend-type: "vxlan"
    flannel.alpha.coreos.com/kube-subnet-manager: "true"
spec:
  #Replace Subnet with your BIG-IP Flannel Subnet
  podCIDR: "10.244.20.0/24"
```

Create the BIG-IP node:

    kubectl create -f bigip-node.yaml
