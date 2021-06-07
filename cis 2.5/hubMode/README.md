# HubMode option when using ConfigMap

A service of type LoadBalancer is the simplest and the fastest way to expose a service inside a Kubernetes cluster to the external world. All you need to-do is specify the service type as type=LoadBalancer in the service definition.

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.4/servicetypelb/diagram/2021-04-27_10-11-10.png)

Demo on YouTube [video]()

Looking at the diagram and Service of type LoadBalancer, the following events occur:

1. CIS will update the service whenever the loadBalancer IP in the service is empty.
2. The IPAM controller assigns an IP address for the loadBalancer: ingress: object from the ip-range based on the ipamlabel specified but the annotation
3. Once the object is updated with the IP address, CIS automatically configures BIG-IP with the External IP address as shown below

#### Example of Service type LoadBalancer shown in the diagram

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    cis.f5.com/ipamLabel: Test
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"cis.f5.com/ipamLabel":"Test"},"labels":{"app":"f5-demo"},"name":"f5-demo","namespace":"default"},"spec":{"ports":[{"name":"f5-demo","port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"f5-demo"},"sessionAffinity":"None","type":"LoadBalancer"},"status":{"loadBalancer":null}}
  creationTimestamp: "2021-04-19T18:05:23Z"
  labels:
    app: f5-demo
  name: f5-demo
  namespace: default
  resourceVersion: "52258409"
  selfLink: /api/v1/namespaces/default/services/f5-demo
  uid: d8336cc3-8611-48d9-bcfc-c3521c45eef1
spec:
  clusterIP: 10.111.131.138
  externalTrafficPolicy: Cluster
  ports:
  - name: f5-demo
    nodePort: 31970
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: f5-demo
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 10.192.75.113

```