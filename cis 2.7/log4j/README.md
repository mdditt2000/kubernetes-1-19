# Protection your Kubernetes Cluster against the Apache Log4j2 Vulnerability

This demo show how to protect your Kubernetes clusters against the Apache Log4j2 vulnerability using F5 CIS and BIG-IP Advanced WAF. CIS is adding the Advanced WAF policy using a Policy CRD from Kubernetes. Policy CRD is a new feature in CIS 2.7


- Policy CRD feature released in CIS 2.7
- Advanced WAF required with the latest signatures 

```
apiVersion: cis.f5.com/v1
kind: Policy
metadata:
  labels:
    f5cr: "true"
  name: policy-mysite
  namespace: default
spec:
  l7Policies:
    waf: /Common/WAF_Policy
  profiles:
    http: /Common/Custom_HTTP
    logProfiles:
      - /Common/Log all requests
```

Associate the Policy CRD to the VirtualServer CRD

```
apiVersion: "cis.f5.com/v1"
kind: VirtualServer
metadata:
  name: vs-myapp
  labels:
    f5cr: "true"
spec:
  # This is an insecure virtual, Please use TLSProfile to secure the virtual
  # check out tls examples to understand more.
  virtualServerAddress: "10.192.75.117"
  virtualServerHTTPSPort: 443
  httpTraffic: redirect
  tlsProfileName: reencrypt-tls
  policyName: policy-mysite
  host: myapp.f5demo.com
  pools:
  - path: /
    service: f5-demo
    servicePort: 443
```

CIS configures BIG-IP with the WAF policy and Log policy as shown below

![policy](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7/log4j/diagram/2021-12-21_16-19-23.png)

For more information please review the Blog and CVE support articles

- Protection against the Apache Log4j2 Vulnerability (CVE-2021-44228) [Protection against the Apache Log4j2 Vulnerability (CVE-2021-44228)](https://www.f5.com/company/blog/protection-against-apache-log4j2-vulnerability)
- K32171392: Apache Log4j2 vulnerability CVE-2021-45046 [K32171392: Apache Log4j2 vulnerability CVE-2021-45046](https://support.f5.com/csp/article/K32171392)