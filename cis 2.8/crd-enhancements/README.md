# CRD Enhancements 

The purpose of this document is to document new CRD enhancements coming to CIS 2.8

- change default persistence profile on BIG-IP

## Persistence Profile

CIS uses AS3 default persistence profile

- VirtualServer uses **cookie**
- TransportServer uses **source-address**

Persistence can be added to the VirtualServer, TransportServer or Policy CRD. Policy CRD will take precedence. Example below show persistence changes added to the VirtualServer, or TransportServer CRD

![vs-ts](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/crd-enhancements/diagram/2022-02-22_10-43-08.png)

Example below show persistence changes added to the Policy CRD and associated to the VirtualServer CRD

![vs-policy](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/crd-enhancements/diagram/2022-02-22_11-05-16.png)

Example below show persistence changes added to the Policy CRD and associated to the TransportServer CRD

![ts-policy](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/crd-enhancements/diagram/2022-02-22_11-04-17.png)

Demo [YouTube](https://www.youtube.com/watch?v=brlNGGJ0sw0)