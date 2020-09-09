##### Initial Setup for non-HA
```
tmsh create auth partition k8s
tmsh create net tunnels vxlan fl-vxlan port 8472 flooding-type none
tmsh create net tunnels tunnel fl-vxlan key 1 profile fl-vxlan local-address 192.168.200.60
tmsh create net self 10.244.20.60 address 10.244.20.60/255.255.0.0 allow-service none vlan fl-vxlan
```