
Debug images : 
- mysql 
oc run -it --rm mysqldebug --restart=Never --image=docker.io/bitnami/mariadb:latest  --command -- bash

network : 
oc run -it --rm dnscheck --restart=Never --image=registry.access.redhat.com/ubi9/ubi -- bash
microdnf install -y bind-utils





Operator :

oc logs -n openshift-operators deploy/external-secrets-operator-controller-manager --tail=300