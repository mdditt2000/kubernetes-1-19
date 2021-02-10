#!/bin/bash

#delete kubernetes nginx-ingress container, authentication and RBAC
kubectl delete -f nginx-config/ns-and-sa.yaml
kubectl delete -f nginx-config/rbac.yaml
kubectl delete -f nginx-config/default-server-secret.yaml
kubectl delete -f nginx-config/nginx-config.yaml
kubectl delete -f nginx-config/ingress-class.yaml
kubectl delete -f nginx-config/nginx-ingress.yaml
kubectl delete -f nginx-config/nginx-service.yaml