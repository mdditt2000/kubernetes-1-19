# CRD Enhancements coming in CIS 2.9

## Configure Load Balancing Method on VirtualServer and TransportServer CRD

The purpose of this document is to DEMO new CRD enhancements coming to CIS 2.9. This document is also a **pre-view** for early testing, and provide some feedback to F5 Product Management

- change default load balancing method used by AS3 to user define. Following options are available via the AS3 schema:

    - dynamic-ratio-node
    - fastest-app-response
    - fastest-node
    - least-connections-member
    - least-connections-node
    - least-sessions
    - observed-member
    - observed-node
    - predictive-member
    - predictive-node
    - ratio-least-connections-membeions-node
    - ratio-member
    - ratio-session
    - round-robin **"default"**
    - weighted-least-connections-member
    - weighted-least-connections-node

## loadBalancingMethod

**loadBalancingMethod** can be added to the VirtualServer, TransportServer CRD. Example below show **loadBalancingMethod** changes added to the VirtualServer CRD and TransportServer CRD

![vs-ts](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.9/loadbalancingmethod/diagram/2022-04-22_15-36-20.png)

Demo [YouTube](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.9/loadbalancingmethod)