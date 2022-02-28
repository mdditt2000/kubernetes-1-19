# Platform-agnostic Container Environments using DNS Failover

Today, organizations are increasingly deploying multiple container environment. Deploying multiple environment can improve availability, isolation and scalability. This user-guide demonstrates how F5 Container Ingress Services (CIS) can automate BIP-IP to provide Ingress services for a multiple container environments.

## OpenShift and Kubernetes Container Environments

In this user-guide, we have deployed an OpenShift and Kubernetes container environments running identical applications. BIG-IP is platform-agnostic, using DNS to distribute traffic between the two container environments using round-robin. This simple but powerful approach enables users the flexibility to complete an container environment proof of concept or migrating applications between environments. Since CIS uses the Kubernetes API the resource definitions for OpenShift and Kubernetes are identical except for the public IPs. Diagram below represents the OpenShift and Kubernetes environments.

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/multi-deployment/diagram/2022-02-28_09-39-06.png)