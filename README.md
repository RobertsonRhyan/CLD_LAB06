# Cloud Computing
## LAB 05 - Kubernetes
### TASK 1 - DEPLOY THE APPLICATION ON A LOCAL TEST CLUSTER
#### DELIVERABLES
Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

> **Frontend displays but cannot create task**
- Front-end : Displays Todos V2 page but cannot create task
- API Pod seems to work : When tested with port forwarding 8081:8081, API returns empty array

Solution : Typo in frontend-pod.yaml


#### Frontend Service
```
$ kubectl describe svc/frontend-svc
W0512 15:23:00.093324    4394 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
Name:                     frontend-svc
Namespace:                default
Labels:                   component=frontend
Annotations:              cloud.google.com/neg: {"ingress":true}
Selector:                 app=todo,component=frontend
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.9.89
IPs:                      10.96.9.89
LoadBalancer Ingress:     34.116.195.249
Port:                     frontend  80/TCP
TargetPort:               8080/TCP
NodePort:                 frontend  30131/TCP
Endpoints:                10.92.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```




