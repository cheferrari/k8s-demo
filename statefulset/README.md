## StatefulSet demo
refer to https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/

#### Create ns,headless svc,sts
```
# kubectl apply -f nginx-with-statefulset.yaml

[root@k8s-node1 statefulset]# kubectl get svc,sts,pods,pvc,pv -n sts-demo 
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/nginx   ClusterIP   None         <none>        80/TCP    96m

NAME                   DESIRED   CURRENT   AGE
statefulset.apps/web   3         3         96m

NAME        READY   STATUS    RESTARTS   AGE
pod/web-0   1/1     Running   0          96m
pod/web-1   1/1     Running   0          95m
pod/web-2   1/1     Running   0          95m

NAME                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/www-web-0   Bound    pvc-f6c5d3d6-cfa3-11e8-b6f8-000c2908d5de   1Gi        RWO            standard       96m
persistentvolumeclaim/www-web-1   Bound    pvc-fc09543a-cfa3-11e8-b6f8-000c2908d5de   1Gi        RWO            standard       95m
persistentvolumeclaim/www-web-2   Bound    pvc-02acd699-cfa4-11e8-b6f8-000c2908d5de   1Gi        RWO            standard       95m

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
persistentvolume/pvc-02acd699-cfa4-11e8-b6f8-000c2908d5de   1Gi        RWO            Retain           Bound    sts-demo/www-web-2   standard                95m
persistentvolume/pvc-f6c5d3d6-cfa3-11e8-b6f8-000c2908d5de   1Gi        RWO            Retain           Bound    sts-demo/www-web-0   standard                95m
persistentvolume/pvc-fc09543a-cfa3-11e8-b6f8-000c2908d5de   1Gi        RWO            Retain           Bound    sts-demo/www-web-1   standard                95m

[root@k8s-node1 statefulset]# dig -t A nginx.sts-demo.svc.cluster.local @10.96.0.10

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7_5.1 <<>> -t A nginx.sts-demo.svc.cluster.local @10.96.0.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42818
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nginx.sts-demo.svc.cluster.local. IN	A

;; ANSWER SECTION:
nginx.sts-demo.svc.cluster.local. 5 IN	A	10.244.1.23
nginx.sts-demo.svc.cluster.local. 5 IN	A	10.244.0.116
nginx.sts-demo.svc.cluster.local. 5 IN	A	10.244.1.24

;; Query time: 0 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Sun Oct 14 21:00:08 CST 2018
;; MSG SIZE  rcvd: 205

[root@k8s-node1 statefulset]# dig -t A web-0.nginx.sts-demo.svc.cluster.local @10.96.0.10

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7_5.1 <<>> -t A web-0.nginx.sts-demo.svc.cluster.local @10.96.0.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32678
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;web-0.nginx.sts-demo.svc.cluster.local.	IN A

;; ANSWER SECTION:
web-0.nginx.sts-demo.svc.cluster.local.	5 IN A	10.244.1.23

;; Query time: 0 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Sun Oct 14 21:00:14 CST 2018
;; MSG SIZE  rcvd: 121
```
#### Delete pod for test rescheduling
```
[root@k8s-node1 statefulset]# kubectl delete pod -l app=nginx
pod "web-0" deleted
pod "web-1" deleted
pod "web-2" deleted
[root@k8s-node1 statefulset]#
```
#### Open another terminal to watch what happend
```
[root@k8s-node1 ~]# kubectl get pods -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          100m
web-1   1/1     Running   0          100m
web-2   1/1     Running   0          99m
web-0   1/1   Terminating   0     103m
web-1   1/1   Terminating   0     103m
web-2   1/1   Terminating   0     103m
web-0   0/1   Terminating   0     103m
web-1   0/1   Terminating   0     103m
web-2   0/1   Terminating   0     103m
web-2   0/1   Terminating   0     103m
web-2   0/1   Terminating   0     103m
web-1   0/1   Terminating   0     103m
web-1   0/1   Terminating   0     103m
web-0   0/1   Terminating   0     103m
web-0   0/1   Terminating   0     103m
web-0   0/1   Pending   0     0s
web-0   0/1   Pending   0     0s
web-0   0/1   ContainerCreating   0     0s
web-0   0/1   ContainerCreating   0     2s
web-0   1/1   Running   0     3s
web-1   0/1   Pending   0     0s
web-1   0/1   Pending   0     0s
web-1   0/1   ContainerCreating   0     0s
web-1   0/1   ContainerCreating   0     0s
web-1   1/1   Running   0     1s
web-2   0/1   Pending   0     0s
web-2   0/1   Pending   0     0s
web-2   0/1   ContainerCreating   0     0s
web-2   0/1   ContainerCreating   0     0s
web-2   1/1   Running   0     1s
```
Pod Management Policies
In Kubernetes 1.7 and later, StatefulSet allows you to relax its ordering guarantees while preserving its uniqueness and identity guarantees via its .spec.podManagementPolicy field.
##### OrderedReady pod management is the default for StatefulSets
##### So until web-0 is Running and Ready,the web-1 just will start deploy.
##### FIELD:    podManagementPolicy <string>
DESCRIPTION:
     podManagementPolicy controls how pods are created during initial scale up,
     when replacing pods on nodes, or when scaling down. The default policy is
     `OrderedReady`, where pods are created in increasing order (pod-0, then
     pod-1, etc) and the controller will wait until each pod is ready before
     continuing. When scaling down, the pods are removed in the opposite order.
     The alternative policy is `Parallel` which will create pods in parallel to
     match the desired scale without waiting, and on scale down will delete all
     pods at once.

#### Verify the web servers continue to serve their hostnames.
```
for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
web-0
web-1
web-2
```
Even though web-0,web-1 and web-2 were rescheduled, they continue to serve their hostnames because the PersistentVolumes associated with their PersistentVolumeClaims are remounted to their volumeMounts. No matter what node web-0and web-1 are scheduled on, their PersistentVolumes will be mounted to the appropriate mount points

#### Scale Up and Scale Down
##### Scale Up, Use command 'kubectl scale'
```
# kubectl scale sts web --replicas=6
statefulset.apps/web scaled
# kubectl get pod -o wide -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE
web-0   1/1     Running   0          46s   10.244.0.122   k8s-node1   <none>
web-1   1/1     Running   0          45s   10.244.1.45    k8s-node2   <none>
web-2   1/1     Running   0          31s   10.244.0.123   k8s-node1   <none>
web-3   0/1   Pending   0     0s    <none>   <none>   <none>
web-3   0/1   Pending   0     0s    <none>   <none>   <none>
web-3   0/1   Pending   0     3s    <none>   k8s-node2   <none>
web-3   0/1   ContainerCreating   0     3s    <none>   k8s-node2   <none>
web-3   0/1   ContainerCreating   0     4s    <none>   k8s-node2   <none>
web-3   1/1   Running   0     4s    10.244.1.46   k8s-node2   <none>
web-4   0/1   Pending   0     0s    <none>   <none>   <none>
web-4   0/1   Pending   0     0s    <none>   <none>   <none>
web-4   0/1   Pending   0     14s   <none>   k8s-node1   <none>
web-4   0/1   ContainerCreating   0     14s   <none>   k8s-node1   <none>
web-4   0/1   ContainerCreating   0     15s   <none>   k8s-node1   <none>
web-4   1/1   Running   0     16s   10.244.0.124   k8s-node1   <none>
web-5   0/1   Pending   0     0s    <none>   <none>   <none>
web-5   0/1   Pending   0     0s    <none>   <none>   <none>
web-5   0/1   Pending   0     0s    <none>   k8s-node2   <none>
web-5   0/1   ContainerCreating   0     0s    <none>   k8s-node2   <none>
web-5   0/1   ContainerCreating   0     0s    <none>   k8s-node2   <none>
web-5   1/1   Running   0     1s    10.244.1.47   k8s-node2   <none>
```
##### Scale Down, Use command 'kubectl patch'
```
# kubectl patch sts web -p '{"spec":{"replicas":3}}'
statefulset.apps/web patched
# kubectl get pod -o wide -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE
web-0   1/1     Running   0          10m     10.244.0.122   k8s-node1   <none>
web-1   1/1     Running   0          10m     10.244.1.45    k8s-node2   <none>
web-2   1/1     Running   0          10m     10.244.0.123   k8s-node1   <none>
web-3   1/1     Running   0          6m13s   10.244.1.46    k8s-node2   <none>
web-4   1/1     Running   0          6m9s    10.244.0.124   k8s-node1   <none>
web-5   1/1     Running   0          5m53s   10.244.1.47    k8s-node2   <none>
web-5   1/1   Terminating   0     6m7s   10.244.1.47   k8s-node2   <none>
web-5   0/1   Terminating   0     6m8s   10.244.1.47   k8s-node2   <none>
web-5   0/1   Terminating   0     6m9s   10.244.1.47   k8s-node2   <none>
web-5   0/1   Terminating   0     6m9s   10.244.1.47   k8s-node2   <none>
web-4   1/1   Terminating   0     6m25s   10.244.0.124   k8s-node1   <none>
web-4   0/1   Terminating   0     6m26s   10.244.0.124   k8s-node1   <none>
web-4   0/1   Terminating   0     6m27s   10.244.0.124   k8s-node1   <none>
web-4   0/1   Terminating   0     6m27s   10.244.0.124   k8s-node1   <none>
web-3   1/1   Terminating   0     6m31s   10.244.1.46   k8s-node2   <none>
web-3   0/1   Terminating   0     6m32s   10.244.1.46   k8s-node2   <none>
web-3   0/1   Terminating   0     6m33s   10.244.1.46   k8s-node2   <none>
web-3   0/1   Terminating   0     6m33s   10.244.1.46   k8s-node2   <none>
```
#### Update Strategy
##### Rolling Update 滚动更新（自动）
逆序自动化更新Pod
###### 自定义滚动更新策略
```
# kubectl explain sts.spec.updateStrategy.rollingUpdate
KIND:     StatefulSet
VERSION:  apps/v1

RESOURCE: rollingUpdate <Object>

DESCRIPTION:
     RollingUpdate is used to communicate parameters when Type is
     RollingUpdateStatefulSetStrategyType.

     RollingUpdateStatefulSetStrategy is used to communicate parameter for
     RollingUpdateStatefulSetStrategyType.

FIELDS:
   partition	<integer>
     Partition indicates the ordinal at which the StatefulSet should be
     partitioned. Default value is 0.
```
若 partition = N, 则 Pod 编号 >= N 的将会更新
###### 实践:
####### patch 打补丁设置 partition=4
```
[root@k8s-node1 statefulset]# kubectl patch sts web -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":4}}}}'
statefulset.apps/web patched
[root@k8s-node1 statefulset]# 
[root@k8s-node1 statefulset]# kubectl describe sts web
···
Replicas:           6 desired | 6 total
Update Strategy:    RollingUpdate
  Partition:        4
···
```
####### 打补丁更新pod镜像或直接set image 更新镜像
```
# kubectl set image sts web nginx=nginx:1.15-alpine
statefulset.apps/web image updated
# kubectl get pod -o wide -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE
web-0   1/1     Running   0          19m   10.244.0.122   k8s-node1   <none>
web-1   1/1     Running   0          19m   10.244.1.45    k8s-node2   <none>
web-2   1/1     Running   0          19m   10.244.0.123   k8s-node1   <none>
web-3   1/1     Running   0          7s    10.244.1.48    k8s-node2   <none>
web-4   1/1     Running   0          6s    10.244.1.49    k8s-node2   <none>
web-5   1/1     Running   0          4s    10.244.0.125   k8s-node1   <none>
web-5   1/1   Terminating   0     4m10s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Terminating   0     4m12s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Terminating   0     4m18s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Terminating   0     4m18s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Pending   0     0s    <none>   <none>   <none>
web-5   0/1   Pending   0     0s    <none>   k8s-node1   <none>
web-5   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-5   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-5   1/1   Running   0     5s    10.244.0.126   k8s-node1   <none>
web-4   1/1   Terminating   0     4m25s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Terminating   0     4m26s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Terminating   0     4m31s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Terminating   0     4m31s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Pending   0     0s    <none>   <none>   <none>
web-4   0/1   Pending   0     1s    <none>   k8s-node2   <none>
web-4   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-4   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-4   1/1   Running   0     5s    10.244.1.50   k8s-node2   <none>
# kubectl describe pod web-3
···
Containers:
  nginx:
    Container ID:   docker://57cbb191c2ad7f4ad2f1ac0cef6341e5022afe6358ad3b5bfe50846111064d30
    Image:          nginx:1.14.0-alpine
···
# kubectl describe pod web-4
···
Containers:
  nginx:
    Container ID:   docker://a9c8de6e699a8aa8b9d80d0a8004460c1bc1a240e83bcadb3d8ca7c96ca16a0e
    Image:          nginx:1.15-alpine
···
```
sts/web 现在共有6个副本Pod，设置spec.updateStrategy.rollingUpdate.partition=4，更新补丁打好后，逆序从web-5更新，更新到web-4（包含web-4）停止，其余的不再更新，更新完后，可以看 web-3 的 image依然为 nginx:1.14.0-alpine，没有更新; web-4 的 image 更新到 nginx:1.15-alpine
####### 这种更新类似于“金丝雀部署”

####### 继续操作，patch 打补丁设置 partition=0
```
# kubectl patch sts web -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":0}}}}'
statefulset.apps/web patched
# kubectl get pod -o wide -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE
web-0   1/1     Running   0          19m   10.244.0.122   k8s-node1   <none>
web-1   1/1     Running   0          19m   10.244.1.45    k8s-node2   <none>
web-2   1/1     Running   0          19m   10.244.0.123   k8s-node1   <none>
web-3   1/1     Running   0          7s    10.244.1.48    k8s-node2   <none>
web-4   1/1     Running   0          6s    10.244.1.49    k8s-node2   <none>
web-5   1/1     Running   0          4s    10.244.0.125   k8s-node1   <none>
web-5   1/1   Terminating   0     4m10s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Terminating   0     4m12s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Terminating   0     4m18s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Terminating   0     4m18s   10.244.0.125   k8s-node1   <none>
web-5   0/1   Pending   0     0s    <none>   <none>   <none>
web-5   0/1   Pending   0     0s    <none>   k8s-node1   <none>
web-5   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-5   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-5   1/1   Running   0     5s    10.244.0.126   k8s-node1   <none>
web-4   1/1   Terminating   0     4m25s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Terminating   0     4m26s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Terminating   0     4m31s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Terminating   0     4m31s   10.244.1.49   k8s-node2   <none>
web-4   0/1   Pending   0     0s    <none>   <none>   <none>
web-4   0/1   Pending   0     1s    <none>   k8s-node2   <none>
web-4   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-4   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-4   1/1   Running   0     5s    10.244.1.50   k8s-node2   <none>
web-3   1/1   Terminating   0     12m   10.244.1.48   k8s-node2   <none>
web-3   0/1   Terminating   0     12m   10.244.1.48   k8s-node2   <none>
web-3   0/1   Terminating   0     12m   10.244.1.48   k8s-node2   <none>
web-3   0/1   Terminating   0     12m   10.244.1.48   k8s-node2   <none>
web-3   0/1   Pending   0     0s    <none>   <none>   <none>
web-3   0/1   Pending   0     0s    <none>   k8s-node2   <none>
web-3   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-3   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-3   1/1   Running   0     2s    10.244.1.51   k8s-node2   <none>
web-2   1/1   Terminating   0     31m   10.244.0.123   k8s-node1   <none>
web-2   0/1   Terminating   0     31m   10.244.0.123   k8s-node1   <none>
web-2   0/1   Terminating   0     31m   10.244.0.123   k8s-node1   <none>
web-2   0/1   Terminating   0     31m   10.244.0.123   k8s-node1   <none>
web-2   0/1   Pending   0     0s    <none>   <none>   <none>
web-2   0/1   Pending   0     0s    <none>   k8s-node1   <none>
web-2   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-2   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-2   1/1   Running   0     0s    10.244.0.127   k8s-node1   <none>
web-1   1/1   Terminating   0     31m   10.244.1.45   k8s-node2   <none>
web-1   0/1   Terminating   0     31m   10.244.1.45   k8s-node2   <none>
web-1   0/1   Terminating   0     32m   10.244.1.45   k8s-node2   <none>
web-1   0/1   Terminating   0     32m   10.244.1.45   k8s-node2   <none>
web-1   0/1   Pending   0     0s    <none>   <none>   <none>
web-1   0/1   Pending   0     0s    <none>   k8s-node2   <none>
web-1   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-1   0/1   ContainerCreating   0     1s    <none>   k8s-node2   <none>
web-1   1/1   Running   0     2s    10.244.1.52   k8s-node2   <none>
web-0   1/1   Terminating   0     32m   10.244.0.122   k8s-node1   <none>
web-0   0/1   Terminating   0     32m   10.244.0.122   k8s-node1   <none>
web-0   0/1   Terminating   0     32m   10.244.0.122   k8s-node1   <none>
web-0   0/1   Terminating   0     32m   10.244.0.122   k8s-node1   <none>
web-0   0/1   Pending   0     0s    <none>   <none>   <none>
web-0   0/1   Pending   0     0s    <none>   k8s-node1   <none>
web-0   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-0   0/1   ContainerCreating   0     0s    <none>   k8s-node1   <none>
web-0   1/1   Running   0     1s    10.244.0.128   k8s-node1   <none>
```
可以看到接着上次，从web-3开始更新，一致更新到web-0 ,所有Pod均被更新了image

