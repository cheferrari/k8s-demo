#### MySQL StatefulSet Demo
refer to https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/

##### Create Service,ConfigMap,StatefulSet
```
kubectl apply -f mysql-service.yaml
kubectl apply -f mysql-configmap.yaml
kubectl get pod -w -l app=mysql
kubectl apply -f mysql-statefulset.yaml
```
