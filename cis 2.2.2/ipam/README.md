# F5 IPAM Controller for CIS 2.2.2

The F5 IPAM Controller is a Docker container that allocates IP addresses from an static list for hostnames. The F5 IPAM Controller watches CRD resources and consumes the hostnames within each resource.

The Controller can:

* Allocate IP address from static IP address pool based on the CIDR mentioned in a Kubernetes resource

* F5 IPAM Controller decides to allocate the IP from the respective IP address pool for the hostname specified in the virtualserver custom resource

**Note** The idea here is that we will support CRD, Type LB and possibly  route/ingress in the future

## F5 IPAM Deploy Configuration Options

* --orchestration=kubernetes

The orchestration parameter holds the orchestration environment i.e. Kubernetes.

* --ip-range="10.192.75.111/24-10.192.75.115/24"28"

ip-range parameter holds the IP address ranges and from this range, it creates a pool of IP address range which gets allocated to the corresponding hostname in the virtual server CRD.

* --log-level=debug

```
- args:
    - --orchestration=kubernetes
    - --ip-range="10.192.75.111/24-10.192.75.115/24"
    - --log-level=DEBUG
```

### Step 1

Deploy the RBAC for F5 IPAM Controller and F5 IPAM Controller

```
kubectl create -f f5-ipam-rbac.yaml
kubectl create -f f5-ipam-rbac.yaml
```
Files located at