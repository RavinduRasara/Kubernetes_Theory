
&nbsp;

**1.Start minikube**

```
Ravilinux:~$ minikube start

Ravilinux:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

ravindu@Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   35s   v1.35.1

Ravilinux:~$ kubectl get pods
No resources found in default namespace.

Ravilinux:~$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   62s
kube-node-lease   Active   62s
kube-public       Active   62s
kube-system       Active   62s

Ravilinux:~$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   91s

```

&nbsp;

```
ravindu@Ravilinux:~/Desktop/Kubernetes_Theory/6.Kubernetes-Services$ ls
devops  Dockerfile  requirements.txt

```

&nbsp;

**2\. Dockerfile and push docker image to dockerhub**

```
FROM ubuntu

WORKDIR /app

COPY requirements.txt /app
COPY devops /app

RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    pip install --break-system-packages -r requirements.txt

ENTRYPOINT ["python3"]
CMD ["manage.py", "runserver", "0.0.0.0:8000"]
                                                     
```

```
Kubernetes-Services$ docker login -u ravi943

i Info → A Personal Access Token (PAT) can be used instead.
         To create a PAT, visit https://app.docker.com/settings
         
         
Password: 

WARNING! Your credentials are stored unencrypted in '/home/ravindu/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded

```

```bash
Kubernetes-Services$ docker push ravi943/python-sample-app-demo:v1

The push refers to repository [docker.io/ravi943/python-sample-app-demo]
6e86424a6650: Pushed 
69798425136b: Pushed 
7a29cceca459: Pushed 
ce189430bfa6: Pushing [============>                                      ]  52.43MB/204.6MB

```

&nbsp;

**3\. Create deployment.yml**

```
Kubernetes-Services$ vim deployment.yml
```

**deployment.yml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-sample-app
  labels:
    app: python-sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-sample-app
  template:
    metadata:
      labels:
        app: python-sample-app
    spec:
      containers:
      - name: python-app
        image: ravi943/python-sample-app-demo:v1
        ports:
        - containerPort: 8000

```

&nbsp;

```bash
Kubernetes-Services$ kubectl apply -f deployment.yml

deployment.apps/python-sample-app unchanged

```

**4.Checking deploy and pods**

```
Kubernetes-Services$ kubectl get deploy

NAME                READY   UP-TO-DATE   AVAILABLE   AGE
python-sample-app   2/2     2            2           175m

Kubernetes-Services$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
python-sample-app-5f95f8b87d-gpcmm   1/1     Running   0          176m

```

```
Kubernetes-Services$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-gpcmm   1/1     Running   0          3h18m   10.244.0.6   minikube   <none>           <none>
python-sample-app-5f95f8b87d-p9m9q   1/1     Running   0          3h18m   10.244.0.5   minikube   <none>           <none>

```

&nbsp;

```bash
Kubernetes-Services$ kubectl get pods -v=7

I0425 23:09:31.893645   81066 loader.go:405] Config loaded from file:  /home/ravindu/.kube/config
I0425 23:09:31.894457   81066 envvar.go:172] "Feature gate default state" feature="ClientsPreferCBOR" enabled=false
I0425 23:09:31.894482   81066 envvar.go:172] "Feature gate default state" feature="InOrderInformers" enabled=true
I0425 23:09:31.894499   81066 envvar.go:172] "Feature gate default state" feature="InOrderInformersBatchProcess" enabled=true
I0425 23:09:31.894515   81066 envvar.go:172] "Feature gate default state" feature="InformerResourceVersion" enabled=true
I0425 23:09:31.894526   81066 envvar.go:172] "Feature gate default state" feature="WatchListClient" enabled=true
I0425 23:09:31.894520   81066 cert_rotation.go:141] "Starting client certificate rotation controller" logger="tls-transport-cache"
I0425 23:09:31.894549   81066 envvar.go:172] "Feature gate default state" feature="ClientsAllowCBOR" enabled=false
I0425 23:09:31.901342   81066 round_trippers.go:527] "Request" verb="GET" url="https://127.0.0.1:43993/api/v1/namespaces/default/pods?limit=500" headers=<
    Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json
    User-Agent: kubectl/v1.35.3 (linux/amd64) kubernetes/6c1cd99
 >
I0425 23:09:31.932899   81066 round_trippers.go:632] "Response" status="200 OK" milliseconds=31
NAME                                 READY   STATUS    RESTARTS   AGE
python-sample-app-5f95f8b87d-gpcmm   1/1     Running   0          3h21m
python-sample-app-5f95f8b87d-p9m9q   1/1     Running   0          3h21m

```