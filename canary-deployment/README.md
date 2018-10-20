## 金丝雀部署示例
### 1、my-web.yaml
两个deployment分别对应版本v1和v2，但他们都有共同的标签 app=webserver，那么service 通过这个标签可以把两个版本的Pod都选中作为后端  
v1有三个副本，v2副本有一个，service采用轮询策略，那么4个请求中，会分发到v1三个，v2一个，以此做到流量分发的效果  
#### k8s-canary 示意图
![k8s-canary](https://github.com/cheferrari/k8s-ingress-controller-demo/blob/master/Traefik/img/k8s-canary.png)
```
# cat my-web.yaml 
apiVersion: v1
kind: Namespace
metadata:
 name: k8s-canary-deploy
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v1
  namespace: k8s-canary-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
      version: webserver-version-1
  template:
    metadata:
      labels:
        app: webserver
        version: webserver-version-1
    spec:
      containers:
      - name: webserver-version-1
        image: ferrariche/myweb:v1
        resources:
          requests:
            cpu: 0.1
            memory: 150
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v2
  namespace: k8s-canary-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webserver
      version: webserver-version-2
  template:
    metadata:
      labels:
        app: webserver
        version: webserver-version-2
    spec:
      containers:
      - name: webserver-version-2
        image: ferrariche/myweb:v2
        resources:
          requests:
            cpu: 0.1
            memory: 150
---
apiVersion: v1
kind: Service
metadata:
  name: webserver
  namespace: k8s-canary-deploy
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: webserver
```
### 2、访问服务测试
#### 首先获取service的IP
```
[root@k8s-node1 ~]# kubectl get svc
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
webserver   ClusterIP   10.106.194.107   <none>        80/TCP    14m
```
#### 使用 curl 循环访问服务
```
# while true; do curl 10.106.194.107; sleep 1; done
<h1>My websever Version: v2!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v2!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v2!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v2!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v1!</h1>
<h1>My websever Version: v2!</h1>
```
