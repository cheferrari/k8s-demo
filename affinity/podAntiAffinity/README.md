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
