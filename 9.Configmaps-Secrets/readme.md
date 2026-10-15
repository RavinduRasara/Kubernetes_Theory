

```
9.Configmaps-Secrets$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      26

9.Configmaps-Secrets$ vim cm.yaml
```

**cm.yaml**
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  db-port: "3306"
```

```
9.Configmaps-Secrets$ kubectl apply -f cm.yaml
configmap/test-cm created

9.Configmaps-Secrets$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      26d
test-cm            1      7s
```

```
9.Configmaps-Secrets$ kubectl describe cm test-cm
Name:         test-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
db-port:
----
3306

BinaryData
====

Events:  <none>
```

**Interactive terminal session inside your running container** 
```
9.Configmaps-Secrets$ kubectl exec -it harry-deployment-87d4bb4d6-b72gh -- /bin/sh
/usr/share/nginx/html # 
```

`kubectl exec`: Tells Kubernetes to run a command inside a container.

**looking environment variable in pods**
```
/usr/share/nginx/html # env | grep db
/usr/share/nginx/html #
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: harry-deployment
  labels:
    app: harry-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: harry-app
  template:
    metadata:
      labels:
        app: harry-app
    spec:
      containers:
      - name: harry-container
        image: ravi943/harry-app:1.0
        env:
         - name: DB-PORT
           valueFrom:
            configMapKeyRef:
             name: test-cm
             key: db-port
        ports:
        - containerPort: 80
```

```
9.Configmaps-Secrets$ kubectl apply -f deployment1.yaml
deployment.apps/harry-deployment configured
```

```
9.Configmaps-Secrets$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE       NOMINATED NODE   READINESS GATES
harry-deployment-5c5c8c7947-56jqs   1/1     Running   0          4h5m   10.244.0.72   minikube   <none>           <none>
harry-deployment-5c5c8c7947-ctcjb   1/1     Running   0          4h5m   10.244.0.71   minikube   <none>           <none>
```

To run  command inside a container for  confirm configmap exist inside in pod
```
ravindu@Ravilinux:~/Desktop/Kubernetes_Theory/9.Configmaps-Secrets$ kubectl exec -it harry-deployment-5c5c8c7947-56jqs -- /bin/sh

/usr/share/nginx/html # env | grep DB
DB-PORT=3306
/usr/share/nginx/html # 
```
