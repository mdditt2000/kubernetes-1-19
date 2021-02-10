#!/bin/bash

#create kubernetes bigip container connecter, authentication and RBAC
kubectl create -f nginx-config/ns-and-sa.yaml
kubectl create -f nginx-config/rbac.yaml
kubectl create -f nginx-config/default-server-secret.yaml
kubectl create -f nginx-config/nginx-config.yaml
kubectl create -f nginx-config/ingress-class.yaml
kubectl create -f nginx-config/nginx-ingress.yaml
kubectl create -f nginx-config/nginx-service.yaml