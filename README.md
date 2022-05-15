# Cloud Computing

## LAB 05 - Kubernetes

De Bleser Dimitri, Peer Vincent, Robertson Rhyan

### TASK 1 - DEPLOY THE APPLICATION ON A LOCAL TEST CLUSTER

#### DELIVERABLES

> Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

Frontend displays but cannot create task

- Front-end : Displays Todos V2 page but cannot create task
- API Pod seems to work : When tested with port forwarding 8081:8081, API returns empty array

Solution : Typo in frontend-pod.yaml

##### Objects

###### Frontend Service

```bash
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

###### Frontend Pod

```bash
$ kubectl describe po/frontend
W0512 16:23:23.435813    5360 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
Name:         frontend
Namespace:    default
Priority:     0
Node:         gke-gke-cluster-1-default-pool-332f1c63-d12c/10.186.0.2
Start Time:   Thu, 12 May 2022 13:30:51 +0200
Labels:       app=todo
              component=frontend
Annotations:  <none>
Status:       Running
IP:           10.92.0.4
IPs:
  IP:  10.92.0.4
Containers:
  frontend:
    Container ID:   containerd://7ef993b2b8cea24d7f743dcdc040607c0b93b605b71b8a721b420b735523579b
    Image:          icclabcna/ccp2-k8s-todo-frontend
    Image ID:       docker.io/icclabcna/ccp2-k8s-todo-frontend@sha256:5892b8f75a4dd3aa9d9cf527f8796a7638dba574ea8e6beef49360a3c67bbb44
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 12 May 2022 13:31:16 +0200
    Ready:          True
    Restart Count:  0
    Limits:
      memory:  128Mi
    Requests:
      memory:  128Mi
    Environment:
      API_ENDPOINT_URL:  http://api-svc:8081
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-z5gr8 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-z5gr8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```
