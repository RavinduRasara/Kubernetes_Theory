

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

### **Volumes**
cm.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  db-port: "3306"
```

deployment.yaml
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
        volumeMounts:
          - name: db-connection
            mountPath: /opt
        ports:
        - containerPort: 80
      volumes:
        - name: db-connection
          configMap:
            name: test-cm
```

```
9.Configmaps-Secrets$ kubectl get pods -w
NAME                               READY   STATUS    RESTARTS   AGE
harry-deployment-6c8f9cd79-7kz5w   1/1     Running   0          31s
harry-deployment-6c8f9cd79-x7sds   1/1     Running   0          29s
```

```
9.Configmaps-Secrets$ kubectl exec -it harry-deployment-6c8f9cd79-7kz5w -- /bin/sh
/usr/share/nginx/html # ls /opt
db-port
/usr/share/nginx/html # cat /opt/db-port | more
3306
```

edit configmap file again

cm.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  db-port: "3307"
```

```
9.Configmaps-Secrets$ kubectl apply -f cm.yaml
configmap/test-cm configured

9.Configmaps-Secrets$ kubectl describe cm test-cm
Name:         test-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
db-port:
----
3307


BinaryData
====

Events:  <none>

```

```
9.Configmaps-Secrets$ kubectl exec -it harry-deployment-6c8f9cd79-7kz5w -- /bin/sh
/usr/share/nginx/html # cat /opt/db-port | more
3307
```

it got change . Port is change 3306 to 3307 without restarting pods.

### Secret 

```
9.Configmaps-Secrets$ kubectl create secret generic test-secret --from-literal=db-port="3306"
secret/test-secret created

9.Configmaps-Secrets$ kubectl describe secret test-secret
Name:         test-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
db-port:  4 bytes
```

what kubernetes does is encrypt password. base64
```
apiVersion: v1
data:
  db-port: MzMwNg==
kind: Secret
metadata:
  creationTimestamp: "2026-10-20T15:38:21Z"
  name: test-secret
  namespace: default
  resourceVersion: "329743"
  uid: 4f02397c-4805-42e9-8d71-4d9c109e2ae3
type: Opaque
```
Decode: 
```
9.Configmaps-Secrets$ echo MzMwNg== | base64 --decode
3306
```
in default, kubenetes does not have a good security
