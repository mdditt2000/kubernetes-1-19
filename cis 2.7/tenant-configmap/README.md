# Tenant based AS3 declarations for ConfigMaps

Today CIS sends a unified AS3 declaration with multiple ConfigMaps. If something is incorrect in any of the ConfigMaps, this will stop applying the correct ConfigMaps configuration. This feature enhances the posting of ConfigMaps to BIG-IP

[YouTube Demo](https://youtu.be/rfvu63dEef4)

- Tenant based AS3 declarations for ConfigMaps feature released in CIS 2.7
- This feature will be available only with CIS parameter **--filter-tenants=true**

```
args: [
  # See the k8s-bigip-ctlr documentation for information about
  # all config options
  # https://clouddocs.f5.com/containers/latest/
    "--bigip-username=$(BIGIP_USERNAME)",
    "--bigip-password=$(BIGIP_PASSWORD)",
    "--bigip-url=192.168.200.60",
    "--bigip-partition=k8s",
    "--pool-member-type=cluster",
    "--flannel-name=fl-vxlan",
    "--log-level=INFO",
    "--insecure=true",
    "--manage-configmaps=true",
    "--filter-tenants=true", -------- Add this argument to enable tenant based AS3 declarations for ConfigMaps
```

CRD [repo](https://github.com/mdditt2000/kubernetes-1-19/tree/master/cis%202.7/log4j/crd)