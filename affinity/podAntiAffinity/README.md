## Pod Anti Affinity Demo
第二个pod必须跟第一个pod不在同一个拓扑域
```
[root@k8s-node1 affinity]# kubectl apply -f pod-required-antiaffinity.yaml 
pod/pod-required-antiaffinity-first created
pod/pod-required-antiaffinity-second created
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE
pod-required-antiaffinity-first    1/1     Running   0          9s    10.244.0.134   k8s-node1   <none>
pod-required-antiaffinity-second   1/1     Running   0          9s    10.244.1.61    k8s-node2   <none>
```
可以看到 两个pod 现在互斥，被调度到不同的节点（拓扑域）上了  
这里我们的 topologyKey=kubernetes.io/hostname 即不同的节点就属于不同的拓扑域
#### Pod Anti Affinity rules
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-required-antiaffinity-first
  labels:
    app: webserver
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-required-antiaffinity-second
  labels:
    app: db
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - webserver
        topologyKey: kubernetes.io/hostname
  containers:
  - name: db
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: 
    - "sh"
    - "-c"
    - "sleep 3600"
```
### 给两个节点都打上同一标签 zone=bar，且修改 topologyKey=zone 看看会发生什么
```
[root@k8s-node1 affinity]# kubectl label nodes k8s-node1 zone=bar
node/k8s-node1 labeled
[root@k8s-node1 affinity]# kubectl label nodes k8s-node2 zone=bar
node/k8s-node2 labeled
[root@k8s-node1 affinity]# kubectl get node --show-labels 
NAME        STATUS   ROLES    AGE     VERSION   LABELS
k8s-node1   Ready    master   7d21h   v1.12.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=k8s-node1,node-role.kubernetes.io/master=,zone=bar
k8s-node2   Ready    <none>   4d3h    v1.12.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=k8s-node2,zone=bar
[root@k8s-node1 affinity]# cat pod-required-antiaffinity2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-required-antiaffinity2-first
  labels:
    app: webserver
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-required-antiaffinity2-second
  labels:
    app: db
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - webserver
        topologyKey: zone
  containers:
  - name: db
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: 
    - "sh"
    - "-c"
    - "sleep 3600"
```
#### create previous pod and create new pod
```
[root@k8s-node1 affinity]# kubectl apply -f pod-required-antiaffinity2.yaml 
pod/pod-required-antiaffinity2-first created
pod/pod-required-antiaffinity2-second created
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE
pod-required-antiaffinity2-first    1/1     Running   0          8s    10.244.1.62   k8s-node2   <none>
pod-required-antiaffinity2-second   0/1     Pending   0          8s    <none>        <none>      <none>
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl describe pod pod-required-antiaffinity2-second 
Name:               pod-required-antiaffinity2-second
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               <none>
Labels:             app=db
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"db"},"name":"pod-required-antiaffinity2-second","namespace":...
Status:             Pending
IP:                 
Containers:
  db:
    Image:      busybox:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      sleep 3600
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-w2r2b (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  default-token-w2r2b:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-w2r2b
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling  4s (x6 over 21s)  default-scheduler  0/2 nodes are available: 2 node(s) didn't match pod affinity/anti-affinity, 2 node(s) didn't match pod anti-affinity rules.
```
可以看到第二个Pod无法被调度，因为没有满足Pod反亲和性规则的节点。  
现在不同的拓扑域判定条件是 根据节点标签 zone，现在集群两个节点都打上了 zone=bar 标签，属于同一个拓扑域。  
第一个pod调度部署后，第一个pod就找不到不同的拓扑域节点了，不能被调度，一致处于Pending状态。
