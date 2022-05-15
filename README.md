# Cloud Computing

## LAB 05 - Kubernetes

De Bleser Dimitri, Peer Vincent, Robertson Rhyan

10.05.2022

### TASK 1 - DEPLOY THE APPLICATION ON A LOCAL TEST CLUSTER

#### DELIVERABLES

> Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

Frontend displays but cannot create task

- Front-end : Displays Todos V2 page but cannot create task
- API Pod seems to work : When tested with port forwarding 8081:8081, API returns empty array

Solution : Typo in frontend-pod.yaml

#### Objects

##### Frontend Pod

```bash
Name:         frontend
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Wed, 11 May 2022 17:57:59 +0200
Labels:       app=todo
              component=frontend
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
IPs:
  IP:  172.17.0.4
Containers:
  frontend:
    Container ID:   docker://b42e38f5c5970bdf138ce786275f9993d17480d36b68e555ada3ddcd97c0d61a
    Image:          icclabcna/ccp2-k8s-todo-frontend
    Image ID:       docker-pullable://icclabcna/ccp2-k8s-todo-frontend@sha256:5892b8f75a4dd3aa9d9cf527f8796a7638dba574ea8e6beef49360a3c67bbb44
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 15 May 2022 16:51:59 +0200
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 11 May 2022 17:58:02 +0200
      Finished:     Thu, 12 May 2022 13:21:37 +0200
    Ready:          True
    Restart Count:  1
    Limits:
      memory:  128Mi
    Requests:
      memory:  128Mi
    Environment:
      API_ENDPOINT_URL:  http://api-svc:8081
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7jnfl (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-7jnfl:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  104s  kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulling         103s  kubelet  Pulling image "icclabcna/ccp2-k8s-todo-frontend"
  Normal  Pulled          100s  kubelet  Successfully pulled image "icclabcna/ccp2-k8s-todo-frontend" in 2.17512712s
  Normal  Created         100s  kubelet  Created container frontend
  Normal  Started         100s  kubelet  Started container frontend
```

#### API Service

```bash
Name:              api-svc
Namespace:         default
Labels:            app=todo
                   component=api
Annotations:       <none>
Selector:          app=todo,component=api
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.253.173
IPs:               10.103.253.173
Port:              api  8081/TCP
TargetPort:        8081/TCP
Endpoints:         172.17.0.7:8081
Session Affinity:  None
Events:            <none>
```

### TASK 2 - DEPLOY THE APPLICATION IN KUBERNETES ENGINE

