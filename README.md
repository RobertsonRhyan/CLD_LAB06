# Cloud Computing

## LAB 05 - Kubernetes

De Bleser Dimitri, Peer Vincent, Robertson Rhyan

10.05.2022

### TASK 1 - DEPLOY THE APPLICATION ON A LOCAL TEST CLUSTER

#### Deliverables for Task 1

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

#### Deliverables for Task 2

> Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report (if they are unchanged from the previous task just say so).

No difficulties were encountered during this task. And the objects weren't changed from the previous task expect for the creation of frontend-svc descirbed bellow.

>Take a screenshot of the cluster details from the GKE console. Copy the output of the kubectl describe command to describe your load balancer once completely initialized.

![Cluster Details](/screenshots/task_02_01.png)

Describe Load Balancer :
```bash
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

### TASK 3 - ADD AND EXERCISE RESILIENCE

> Use only 1 instance for the Redis-Server. Why?

We want to have the same data on all frontends, so we only want one DB.
If we wanted to have multiple replicas of redis, we would use a StatefullSet instead of a Deployment.

> What happens if you delete a Frontend or API Pod? How long does it take for the system to react?

A new Pod is created before the original is even terminated as we can see below:

```bash
NAME                                   READY   STATUS    RESTARTS   AGE
api-deployment-86d8969586-bwjjp        1/1     Running   0          9m43s
api-deployment-86d8969586-wxqpv        1/1     Running   0          9m43s
frontend-deployment-785865d54b-4p5zw   1/1     Running   0          5m50s
frontend-deployment-785865d54b-tpjp2   1/1     Running   0          5m50s
redis-deployement-6fbcc669d9-mrjj4     1/1     Running   0          15m
----
frontend-deployment-785865d54b-tpjp2   1/1     Terminating   0          5m88s
frontend-deployment-785865d54b-zfp96   0/1     Pending       0          0s
frontend-deployment-785865d54b-zfp96   0/1     Pending       0          0s
frontend-deployment-785865d54b-zfp96   0/1     ContainerCreating   0          0s
frontend-deployment-785865d54b-tpjp2   0/1     Terminating         0          5m89s
frontend-deployment-785865d54b-zfp96   1/1     Running             0          3s
frontend-deployment-785865d54b-tpjp2   0/1     Terminating         0          5m93s
frontend-deployment-785865d54b-tpjp2   0/1     Terminating         0          5m93s
```

> What happens when you delete the Redis Pod?

A new Pod is created but the data is lost. This is because the data is stored inside the Pod and not in a local or remote Volume, so it isn't persistent.

The API stops working because it's still trying to query the old redis Pod, we need to restart the api pods for it to work again.

