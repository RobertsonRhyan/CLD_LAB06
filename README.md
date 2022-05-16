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

**Objects**

Frontend Pod:

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

API Service:

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

#### Deliverables for Task 3

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

> How can you change the number of instances temporarily to 3?

In the "Deplyoment details" -> "Actions" -> "Scale" -> change 2 to 3.
![Scale Deplyoment](/screenshots/task_03_01.png)

> What autoscaling features are available? Which metrics are used?

Features :

- Minimum number of replicas
- Maximum number of replicas

Metrics :

- CPU Utilization
- Memory Utilization
- Custom metrics
- External metrics

> How can you update a component? (see "Updating a Deployment" in the deployment documentation)

You can use the `kubectl set` command.

```bash
kubectl set image deployment/redis-deployment redis=redis:7.0-alpine
```

or the `kubectl edit` command to edit the Deployment

```bash
kubectl edit deployment/redis-deployment
```

or the GUI via "Deplyoment details" -> "Actions" -> "Rolling update"

Then you can follow the rollout status with:

```bash
$ kubectl rollout status deployment/redis-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           22s
```

> Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

**Objects**

Frontend Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: todo
    component: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
      component: frontend
  template:
    metadata:
      labels:
        app: todo
        component: frontend
    spec:
      containers:
      - name: frontend
        image: icclabcna/ccp2-k8s-todo-frontend
        ports:
        - containerPort: 8080
        env:
        - name: API_ENDPOINT_URL
          value: http://api-svc:8081
```

Frontend Deplyoment Description:

```bash
Name:                   frontend-deployment
Namespace:              default
CreationTimestamp:      Mon, 16 May 2022 11:34:31 +0200
Labels:                 app=todo
                        component=frontend
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=todo,component=frontend
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=frontend
  Containers:
   frontend:
    Image:      icclabcna/ccp2-k8s-todo-frontend
    Port:       8080/TCP
    Host Port:  0/TCP
    Requests:
      cpu:  100m
    Environment:
      API_ENDPOINT_URL:  http://api-svc:8081
    Mounts:              <none>
  Volumes:               <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-deployment-b5669674f (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m32s  deployment-controller  Scaled up replica set frontend-deployment-b5669674f to 2
```


API deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: todo
    component: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
      component: api
  template:
    metadata:
      labels:
        app: todo
        component: api
    spec:
      containers:
      - name: api
        image: icclabcna/ccp2-k8s-todo-api
        ports:
        - containerPort: 8081
        env:
        - name: REDIS_ENDPOINT
          value: redis-svc
        - name: REDIS_PWD
          value: ccp2
```

API Deplyoment Description:

```bash
Name:                   api-deployment
Namespace:              default
CreationTimestamp:      Mon, 16 May 2022 11:34:24 +0200
Labels:                 app=todo
                        component=api
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=todo,component=api
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=api
  Containers:
   api:
    Image:      icclabcna/ccp2-k8s-todo-api
    Port:       8081/TCP
    Host Port:  0/TCP
    Environment:
      REDIS_ENDPOINT:  redis-svc
      REDIS_PWD:       ccp2
    Mounts:            <none>
  Volumes:             <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   api-deployment-86d8969586 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  67s   deployment-controller  Scaled up replica set api-deployment-86d8969586 to 2
```


REDIS Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployement
  labels:
    app: todo
    component: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: todo
      component: redis
  template:
    metadata:
      labels:
        app: todo
        component: redis
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
        args:
        - redis-server 
        - --requirepass ccp2 
        - --appendonly yes
```

REDIS Deployment Description:

```bash
Name:                   redis-deployement
Namespace:              default
CreationTimestamp:      Mon, 16 May 2022 11:34:18 +0200
Labels:                 app=todo
                        component=redis
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=todo,component=redis
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=redis
  Containers:
   redis:
    Image:      redis
    Port:       6379/TCP
    Host Port:  0/TCP
    Args:
      redis-server
      --requirepass ccp2
      --appendonly yes
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   redis-deployement-6fbcc669d9 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m59s  deployment-controller  Scaled up replica set redis-deployement-6fbcc669d9 to 1
```

### SUBTASK 3.3 (OPTIONAL) - PUT AUTOSCALING IN PLACE AND LOAD-TEST IT

#### Deliverables for Task 3.3

> Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

Difficulties
We had 1 difficulty with the Auto Scaling :

![Auto Scaling](/screenshots/task_03_03_01.png)

As per the official Kubernetes documentation, resource request needs to be set.
And so we could go down to 1 replica, we also changed the replicas value.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: todo
    component: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: todo
      component: frontend
  template:
    metadata:
      labels:
        app: todo
        component: frontend
    spec:
      containers:
      - name: frontend
        image: icclabcna/ccp2-k8s-todo-frontend
        ports:
        - containerPort: 8080
        env:
        - name: API_ENDPOINT_URL
          value: http://api-svc:8081
        resources:
          requests:
            cpu: 100m
```

Description:

```bash
Name:                   frontend-deployment
Namespace:              default
CreationTimestamp:      Mon, 16 May 2022 11:42:32 +0200
Labels:                 app=todo
                        component=frontend
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=todo,component=frontend
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=frontend
  Containers:
   frontend:
    Image:      icclabcna/ccp2-k8s-todo-frontend
    Port:       8080/TCP
    Host Port:  0/TCP
    Requests:
      cpu:  100m
    Environment:
      API_ENDPOINT_URL:  http://api-svc:8081
    Mounts:              <none>
  Volumes:               <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-deployment-b5669674f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  94s   deployment-controller  Scaled up replica set frontend-deployment-b5669674f to 1
```

Theses are the Autoscale settings :

![Auto Scaler](/screenshots/task_03_03_02.png)

When load tested with JMeter, we can see the creation of 3 additional Pods, then after the test (about 5min), we can see them beeing terminated

```bash
NAME                                  READY   STATUS    RESTARTS   AGE
api-deployment-86d8969586-5p4jx       1/1     Running   0          3h41m
api-deployment-86d8969586-m9wdb       1/1     Running   0          3h41m
frontend-deployment-b5669674f-tkkff   1/1     Running   0          3m29s
redis-deployement-6fbcc669d9-krvj9    1/1     Running   0          3h41m
frontend-deployment-b5669674f-dkkcr   0/1     Pending   0          0s
frontend-deployment-b5669674f-dkkcr   0/1     Pending   0          0s
frontend-deployment-b5669674f-dbmcl   0/1     Pending   0          0s
frontend-deployment-b5669674f-dbmcl   0/1     Pending   0          0s
frontend-deployment-b5669674f-78577   0/1     Pending   0          0s
frontend-deployment-b5669674f-dkkcr   0/1     ContainerCreating   0          0s
frontend-deployment-b5669674f-78577   0/1     Pending             0          0s
frontend-deployment-b5669674f-dbmcl   0/1     ContainerCreating   0          0s
frontend-deployment-b5669674f-78577   0/1     ContainerCreating   0          0s
frontend-deployment-b5669674f-dbmcl   1/1     Running             0          2s
frontend-deployment-b5669674f-dkkcr   1/1     Running             0          3s
frontend-deployment-b5669674f-78577   1/1     Running             0          5s
frontend-deployment-b5669674f-dbmcl   1/1     Terminating         0          6m31s
frontend-deployment-b5669674f-dkkcr   1/1     Terminating         0          6m31s
frontend-deployment-b5669674f-78577   1/1     Terminating         0          6m31s
frontend-deployment-b5669674f-dkkcr   0/1     Terminating         0          6m32s
frontend-deployment-b5669674f-78577   0/1     Terminating         0          6m32s
frontend-deployment-b5669674f-dbmcl   0/1     Terminating         0          6m32s
frontend-deployment-b5669674f-78577   0/1     Terminating         0          6m43s
frontend-deployment-b5669674f-78577   0/1     Terminating         0          6m43s
frontend-deployment-b5669674f-dkkcr   0/1     Terminating         0          6m43s
frontend-deployment-b5669674f-dkkcr   0/1     Terminating         0          6m43s
frontend-deployment-b5669674f-dbmcl   0/1     Terminating         0          6m44s
frontend-deployment-b5669674f-dbmcl   0/1     Terminating         0          6m44s
```

### TASK 4 - DEPLOY ON IICT KUBERNETES CLUSTER

#### Deliverables for Task 4

> Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report

No issues were encountered, everything worked fine :

```bash
kubectl get all -n l6grp
NAME                                      READY   STATUS    RESTARTS   AGE
pod/api-deployment-86d8969586-4g4tr       1/1     Running   0          110m
pod/api-deployment-86d8969586-pltpb       1/1     Running   0          110m
pod/frontend-deployment-b5669674f-gxxnk   1/1     Running   0          109m
pod/frontend-deployment-b5669674f-swhpl   1/1     Running   0          109m
pod/redis-deployement-6fbcc669d9-ctgfx    1/1     Running   0          111m

NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
service/api-svc        ClusterIP      10.43.118.89   <none>          8081/TCP       73m
service/frontend-svc   LoadBalancer   10.43.115.50   10.193.72.108   80:31536/TCP   111m
service/redis-svc      ClusterIP      10.43.67.70    <none>          6379/TCP       134m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api-deployment        2/2     2            2           110m
deployment.apps/frontend-deployment   2/2     2            2           109m
deployment.apps/redis-deployement     1/1     1            1           111m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/api-deployment-86d8969586       2         2         2       110m
replicaset.apps/frontend-deployment-b5669674f   2         2         2       109m
replicaset.apps/redis-deployement-6fbcc669d9    1         1         1       111m
````

**Objects**

Frontend Deployment Description :

```bash
Name:                   frontend-deployment
Namespace:              l6grp
CreationTimestamp:      Mon, 16 May 2022 09:59:06 +0200
Labels:                 app=todo
                        component=frontend
Annotations:            deployment.kubernetes.io/revision: 1
                        field.cattle.io/publicEndpoints:
                          [{"addresses":["10.193.72.108"],"port":80,"protocol":"TCP","serviceName":"l6grp:frontend-svc","allNodes":false}]
Selector:               app=todo,component=frontend
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=frontend
  Containers:
   frontend:
    Image:      icclabcna/ccp2-k8s-todo-frontend
    Port:       8080/TCP
    Host Port:  0/TCP
    Requests:
      cpu:  100m
    Environment:
      API_ENDPOINT_URL:  http://api-svc:8081
    Mounts:              <none>
  Volumes:               <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-deployment-b5669674f (2/2 replicas created)
Events:          <none>
```

API Deployment Description :

```bash
Name:                   api-deployment
Namespace:              l6grp
CreationTimestamp:      Mon, 16 May 2022 09:58:11 +0200
Labels:                 app=todo
                        component=api
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=todo,component=api
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=api
  Containers:
   api:
    Image:      icclabcna/ccp2-k8s-todo-api
    Port:       8081/TCP
    Host Port:  0/TCP
    Environment:
      REDIS_ENDPOINT:  redis-svc
      REDIS_PWD:       ccp2
    Mounts:            <none>
  Volumes:             <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   api-deployment-86d8969586 (2/2 replicas created)
Events:          <none>
```

REDIS Deployment Description :

```bash
Name:                   redis-deployement
Namespace:              l6grp
CreationTimestamp:      Mon, 16 May 2022 09:58:05 +0200
Labels:                 app=todo
                        component=redis
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=todo,component=redis
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=todo
           component=redis
  Containers:
   redis:
    Image:      redis
    Port:       6379/TCP
    Host Port:  0/TCP
    Args:
      redis-server
      --requirepass ccp2
      --appendonly yes
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   redis-deployement-6fbcc669d9 (1/1 replicas created)
Events:          <none>
```
