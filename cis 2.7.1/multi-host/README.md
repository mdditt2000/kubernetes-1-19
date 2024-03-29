# Multi-host Support for VirtualServer CRD

Multi-host feature allows CIS to support a single HTTP VirtualServer on BIG-IP for different hostnames. This is very similar to how OpenSHift routes work today. The benefit for using the multi-host feature is re-using the public IP Address on BIG-IP. This helps when Public IP addresses are limited. As you can see from the example below. 

```
apiVersion: "cis.f5.com/v1"
kind: VirtualServer
metadata:
  name: myapp-virtual-server
  labels:
    f5cr: "true"
spec:
  host: myapp.f5demo.com
  virtualServerAddress: "10.192.75.111"
  hostGroup: "f5demo"
  pools:
  - monitor:
      interval: 20
      recv: ""
      send: /
      timeout: 10
      type: http
    path: /myapp
    service: f5-demo
    servicePort: 80
---
apiVersion: "cis.f5.com/v1"
kind: VirtualServer
metadata:
  name: mysite-virtual-server
  labels:
    f5cr: "true"
spec:
  host: mysite.f5demo.com
  virtualServerAddress: "10.192.75.111"
  hostGroup: "f5demo"
  - monitor:
      interval: 20
      recv: ""
      send: /
      timeout: 10
      type: http
    path: /mysite
    service: f5-demo
    servicePort: 80
```