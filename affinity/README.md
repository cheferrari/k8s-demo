## node affinity demo
#### Create Pod with-node-affinity
```
[root@k8s-node1 affinity]# kubectl apply -f pod-with-node-affinity.yaml 
pod/with-node-affinity created
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl get pods 
NAME                                READY   STATUS    RESTARTS   AGE
busybox-bd8fb7cbd-s4vd9             1/1     Running   60         4d5h
nginx-deployment-5947c4dd86-84x5j   1/1     Running   1          4d
nginx-deployment-5947c4dd86-j4mvj   1/1     Running   1          4d
nginx-deployment-5947c4dd86-rzmzq   1/1     Running   1          4d
with-node-affinity                  0/1     Pending   0          10s
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl describe pod with-node-affinity 
Name:               with-node-affinity
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               <none>
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"with-node-affinity","namespace":"default"},"spec":{"affinity":{"nodeA...
Status:             Pending
IP:                 
Containers:
  nginx-with-node-affinity:
    Image:        nginx:alpine
    Port:         80/TCP
    Host Port:    0/TCP
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
  Warning  FailedScheduling  2s (x6 over 20s)  default-scheduler  0/2 nodes are available: 2 node(s) didn't match node selector.
```
#### 给 k8s-node2 打标签 zone=bar
```
[root@k8s-node1 affinity]# kubectl label nodes k8s-node2 zone=bar
node/k8s-node2 labeled
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
busybox-bd8fb7cbd-s4vd9             1/1     Running   60         4d5h
nginx-deployment-5947c4dd86-84x5j   1/1     Running   1          4d
nginx-deployment-5947c4dd86-j4mvj   1/1     Running   1          4d
nginx-deployment-5947c4dd86-rzmzq   1/1     Running   1          4d
with-node-affinity                  1/1     Running   0          119s
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl describe pod with-node-affinity 
Name:               with-node-affinity
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node2/192.168.75.152
Start Time:         Tue, 16 Oct 2018 17:53:41 +0800
Labels:             <none>
Annotations:        cni.projectcalico.org/podIP: 10.244.1.58/32
                    kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"with-node-affinity","namespace":"default"},"spec":{"affinity":{"nodeA...
Status:             Running
IP:                 10.244.1.58
Containers:
  nginx-with-node-affinity:
    Container ID:   docker://1d804b49983485197d6a0df567b75acf45677cf61f299a9b9f1686e60af2bb17
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:fd0361ff0882d63eec241705ba169d83c042bf27f8b568aedd131c2ab97246f0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 16 Oct 2018 17:53:42 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-w2r2b (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
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
  Type     Reason            Age                  From                Message
  ----     ------            ----                 ----                -------
  Warning  FailedScheduling  20s (x24 over 2m8s)  default-scheduler   0/2 nodes are available: 2 node(s) didn't match node selector.
  Normal   Scheduled         15s                  default-scheduler   Successfully assigned default/with-node-affinity to k8s-node2
  Normal   Pulled            14s                  kubelet, k8s-node2  Container image "nginx:alpine" already present on machine
  Normal   Created           14s                  kubelet, k8s-node2  Created container
  Normal   Started           14s                  kubelet, k8s-node2  Started container
```
打上标签后 Pod 可以调度到 node2 上了，pod 状态从 Pending 变成 Running
