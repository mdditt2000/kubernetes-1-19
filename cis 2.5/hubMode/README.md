# HubMode option when using ConfigMap

**Note Preview only until CIS 2.5 release in Mid July**

HubMode expands on current ConfigMap implementation in CIS using the AS3 API. One of key strength of CIS is it can help multiple team collaborate better together. With microservices architecture we are seeing organizations create dedicate team that combine network and system personas. This allegement request that network engineers (NetOps) be added to these teams. F5 CIS ConfigMap is a perfect fit as it can accelerates the developer experience and workflow tooling.
 
Looking at the diagram below, we see the current Kubernetes personas (left in the picture) and “introduction of Operators” (the right), where different personas are responsible for configuration of the platform. Infrastructure Provider responsible for Ingress "ConfigMap" while Cluster Operator and or Application Developers responsible for Pod Deployments/Services. These personas are mostly working in different projects and or namespaces. CIS maps perfectly to both personas with the introduction on HubMode using ConfigMaps. With HubMode CIS can better assist NetOps transform from imperative to declarative using AS3 declarations, no matter of the mode. 

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.5/hubMode/diagram/2021-06-07_12-13-34.png)

CIS maps perfectly to both roles. With HubMode CIS can better assist NetOps transform from imperative to declarative using AS3 declarations, no matter of the mode. In this example the Infrastructure Provider is creating the ConfigMap in the default namespace while the Application Developers is creating the Pod Deployments/Services in n1 namespace. Using HubMode, CIS will detect the associated endpoints using the service discovery labels regards of the namespace. This is consistent with CIS 1.x. 

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.5/hubMode/diagram/2021-06-07_13-28-57.png)

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