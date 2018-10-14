## StatefulSet demo
refer to https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/

#### create ns,headless svc,sts
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
