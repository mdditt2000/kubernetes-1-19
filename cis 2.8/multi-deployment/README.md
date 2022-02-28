# Platform-agnostic Container Environments using DNS Failover

Today, organizations are increasingly deploying multiple container environment. Deploying multiple environment can improve availability, isolation and scalability. This user-guide demonstrates how F5 Container Ingress Services (CIS) can automate BIP-IP to provide Ingress services for a multiple platform-agnostic container environments.

## OpenShift and Kubernetes Container Environments

In this user-guide, we have deployed an OpenShift and Kubernetes container environments running identical applications. BIG-IP is platform-agnostic, using DNS to distribute traffic between the two container environments. This simple but powerful approach enables users the flexibility to complete an container environment proof of concept or migrating applications between environments. Since CIS uses the Kubernetes API the resource definitions for OpenShift and Kubernetes are identical except for the public IPs. Diagram below represents the OpenShift and Kubernetes environments.

![architecture](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/multi-deployment/diagram/2022-02-28_09-39-06.png)

Demo on YouTube [video](Coming soon)

### Environment parameters

* Configure BIG-IP DNS iQuery so that BIG-IP systems can communicate with each other for datacenter **ocp** and **k8s**

![iQuery](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/multi-deployment/diagram/2022-02-28_11-11-06.png)

## OpenShift Container Environments

### Step 1: Deploy CIS

In this user-guide CIS is deployed using a manifest. CIS can also be deployed using the CIS Operator from the OpenShift dashboard. Follow user-guide to deploy CIS using the Operator CIS [Operator CIS](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/operator#readme)



Add the following parameters to THE CIS deployment

* --custom-resource-mode=true - Configure CIS to watch for CRDs. ExternalDNS is not currently supported using OpenShift Routes
* --bigip-partition=OpenShift - CIS uses BIG-IP tenant OpenShift to manage CRDs
* --openshift-sdn-name=/Common/openshift_vxlan - CNI policy on BIG-IP to connect to the PODs in OpenShift

```
args: [
  # See the k8s-bigip-ctlr documentation for information about
  # all config options
  # https://clouddocs.f5.com/containers/latest/
  "--bigip-username=$(BIGIP_USERNAME)",
  "--bigip-password=$(BIGIP_PASSWORD)",
  "--bigip-url=10.192.125.60",
  "--bigip-partition=OpenShift",
  "--namespace=default",
  "--pool-member-type=cluster",
  "--openshift-sdn-name=/Common/openshift_vxlan",
  "--insecure=true",
  "--custom-resource-mode=true",
  "--as3-validation=true",
  "--log-as3-response=true",
]
```

Deploy CIS in OpenShift

```
oc create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
oc create -f bigip-ctlr-clusterrole.yaml
oc create -f f5-bigip-ctlr-deployment.yaml
```

cis-deployment [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.8/multi-deployment/ocp/cis/cis-deployment)