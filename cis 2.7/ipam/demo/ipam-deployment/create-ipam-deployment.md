#!/bin/bash

#create ipam controller authentication RBAC
kubectl create -f f5-ipam-rbac.yaml
kubectl create -f f5-ipam-persitentvolume.yaml
kubectl create -f f5-ipam-deployment.yaml

IPAM CRD schema is not required. CIS creates the CRD