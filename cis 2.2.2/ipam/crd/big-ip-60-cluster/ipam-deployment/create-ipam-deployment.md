#!/bin/bash

#create ipam controller authentication RBAC
kubectl create -f rbac.yaml
kubectl create -f ipam-deployment.yaml