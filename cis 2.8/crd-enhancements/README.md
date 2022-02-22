# CRD Enhancements 

The purpose of this document is to document new CRD enhancements coming to CIS 2.8

- change default persistence profile on BIG-IP

## Persistence Profile

CIS uses AS3 default persistence profile

- VirtualServer uses **cookie**
- TransportServer uses **source-address**

Persistence profile can be added to the VirtualServer, TransportServer or Policy CRD. Policy CRD will take precedence

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.7.1/per-application-failover/diagram/2022-01-31_14-03-56.png)

Demo [YouTube]()