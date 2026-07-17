
## 6. Kubernetes Services

### Kubernetes Service

**Nodes**
```
Kubernetes-Services$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   29h   v1.35.1
```

**Pods**
```
Kubernetes-Services$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS      AGE
python-sample-app-5f95f8b87d-bj6nh   1/1     Running   1 (25h ago)   29h
python-sample-app-5f95f8b87d-fqvsx   1/1     Running   1 (25h ago)   29h
```


**Services**
```
Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP     PORT(S)
kubernetes                  ClusterIP   10.96.0.1      443/TCP
python-django-app-service   NodePort    10.99.32.95    80:30007/TCP
```

**Deployment**
```
Kubernetes-Services$ kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
python-sample-app   2/2     2            2           29h

**Services**
```
Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP     PORT(S)
kubernetes                  ClusterIP   10.96.0.1      443/TCP
python-django-app-service   NodePort    10.99.32.95    80:30007/TCP
```

**Deployment**
```
Kubernetes-Services$ kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
python-sample-app   2/2     2            2           29h
