## Escalation Checklist

To effectively troubleshoot F5 CIS please provide the following information

Environment

- CIS Version: 
- AS3 Version:
- Kubernetes or OpenShift version:
- What CNI: Flannel or Calico

Ingress Resources

- What Ingress Resource is being used

    * Routes
    * ConfigMap
    * Ingress
    * CRDs

- What files/information/logs to gather

    * CIS deployment yaml
    * Associated Service definition along with resource definition
    * Kubernetes or OpenShift logs from the CIS deployment
    * Any AS3 error displayed in the CIS pod logs or restjavad logs on BIG-IP


