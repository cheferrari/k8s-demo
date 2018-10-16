## Pod Affinity/antiAffinity
### Pod Affinity Demo
the second pod "db" want to co-located with the first pod nginx  
当第一个pod被调度后，第二个pod设置了硬性pod亲和性规则（表明我跟那些pod亲和），意思就是第二个pod要跟随第一个pod部署在同一个拓扑域“topology domain”中（这就是所谓的节点亲和）  
spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution.labelSelector 就是选出来具有这些标签的pod，说明我跟这些pod亲和  
调度的时候把我调度到跟具有这些标签的pod 所在的拓扑域。  
那么怎么判定那些节点属于同一拓扑域呢？用 "node label" 即节点上的标签判定，将具有相同节点标签的节点判定属于同一个“拓扑域”  
这里我们集群只有两个节点，我们用 Node 默认 label "kubernetes.io/hostname" 判定不同的拓扑域  
即：k8s-node1 和 k8s-node2 属于两个拓扑域
```
[root@k8s-node1 affinity]# kubectl apply -f pod-required-affinity.yaml 
pod/pod-required-affinity-first created
pod/pod-required-affinity-second created
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE
pod-required-affinity-first    1/1     Running   0          13s   10.244.1.59   k8s-node2   <none>
pod-required-affinity-second   1/1     Running   0          13s   10.244.1.60   k8s-node2   <none>
[root@k8s-node1 affinity]# 
[root@k8s-node1 affinity]# kubectl describe pod pod-required-affinity-second 
Name:               pod-required-affinity-second
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node2/192.168.75.152
Start Time:         Tue, 16 Oct 2018 19:18:19 +0800
Labels:             app=db
Annotations:        cni.projectcalico.org/podIP: 10.244.1.60/32
                    kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"db"},"name":"pod-required-affinity-second","namespace":"defa...
Status:             Running
IP:                 10.244.1.60
Containers:
  db:
    Container ID:  docker://e316438be08ea638b25d7198c2da6f2aaab06cf86b51b1a8859af2a6afef86b5
    Image:         busybox:latest
    Image ID:      docker-pullable://busybox@sha256:2a03a6059f21e150ae84b0973863609494aad70f0a80eaeb64bddd8d92465812
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 3600
    State:          Running
      Started:      Tue, 16 Oct 2018 19:18:20 +0800
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
  Type    Reason     Age   From                Message
  ----    ------     ----  ----                -------
  Normal  Scheduled  2m8s  default-scheduler   Successfully assigned default/pod-required-affinity-second to k8s-node2
  Normal  Pulled     2m7s  kubelet, k8s-node2  Container image "busybox:latest" already present on machine
  Normal  Created    2m7s  kubelet, k8s-node2  Created container
  Normal  Started    2m7s  kubelet, k8s-node2  Started container
```
pod affinity 规则：
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-required-affinity-first
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
  name: pod-required-affinity-second
  labels:
    app: db
spec:
  affinity:
    podAffinity:
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
可以看到second pod 设置了亲和性规则，表明它亲和具有这些标签(app In webserver)的pod, second pod说：请调度的时候把我调度到这些pod所在的拓扑域
