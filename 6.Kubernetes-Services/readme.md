
&nbsp;

**1.Start minikube**

```
ravindu@Ravilinux:~$ minikube start
😄  minikube v1.38.1 on Ubuntu 24.04
✨  Using the docker driver based on existing profile
💨  For improved Docker performance, enable the overlay Linux kernel module using 'modprobe overlay'

⛔  Requested memory allocation (1870MB) is less than the recommended minimum 1900MB. Deployments may fail.

👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.50 ...
🤷  docker "minikube" container is missing, will recreate.
🔥  Creating docker container (CPUs=2, Memory=1870MB) ...
🐳  Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

ravindu@Ravilinux:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

ravindu@Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   13m   v1.35.1

ravindu@Ravilinux:~$ kubectl get pods
No resources found in default namespace.

ravindu@Ravilinux:~$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   13m

ravindu@Ravilinux:~$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   14m
kube-node-lease   Active   14m
kube-public       Active   14m
kube-system       Active   14m


```

**Project Folder**

```
6.Kubernetes-Services$ ls
deployment.yml  devops  Dockerfile  readme.md  requirements.txt

```

&nbsp;

**2\. Build Docker image**

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

**2.1 Login dockerHub**

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

**2.2 Push docker image to docker hub**

```bash
Kubernetes-Services$ docker push ravi943/python-sample-app-demo:v1

The push refers to repository [docker.io/ravi943/python-sample-app-demo]
6e86424a6650: Pushed 
69798425136b: Pushed 
7a29cceca459: Pushed 
ce189430bfa6: Pushing [============>                                      ]  52.43MB/204.6MB

```

**2.3 Build docker image**

```
docker build -t ravi943/python-sample-app-demo:v1 .
```

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

**4\. Created the deployment**

```bash
Kubernetes-Services$ kubectl apply -f deployment.yml
deployment.apps/python-sample-app created

```

**5\. Checking deploy and pods**

```
6.Kubernetes-Services$ kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
python-sample-app   2/2     2            2           118m

6.Kubernetes-Services$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
python-sample-app-5f95f8b87d-g7hgr   1/1     Running   0          118m
python-sample-app-5f95f8b87d-ln6n5   1/1     Running   0          118m

6.Kubernetes-Services$ kubectl get rs
NAME                           DESIRED   CURRENT   READY   AGE
python-sample-app-5f95f8b87d   2         2         2       118m



```

```
6.Kubernetes-Services$ kubectl get pods -o wide

NAME                                 READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-g7hgr   1/1     Running   0          121m   10.244.0.4   minikube   <none>           <none>
python-sample-app-5f95f8b87d-ln6n5   1/1     Running   0          121m   10.244.0.5   minikube   <none>           <none>

```

&nbsp;

**6\. Delete pods and get new pods automatically**

```
6.Kubernetes-Services$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-g7hgr   1/1     Running   0          125m   10.244.0.4   minikube   <none>           <none>
python-sample-app-5f95f8b87d-ln6n5   1/1     Running   0          125m   10.244.0.5   minikube   <none>           <none>

6.Kubernetes-Services$ kubectl delete pod python-sample-app-5f95f8b87d-ln6n5
pod "python-sample-app-5f95f8b87d-ln6n5" deleted from default namespace

6.Kubernetes-Services$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-b4b4v   1/1     Running   0          16s    10.244.0.6   minikube   <none>           <none>
python-sample-app-5f95f8b87d-g7hgr   1/1     Running   0          126m   10.244.0.4   minikube   <none>           <none>

```

```
ravindu@Ravilinux:~/Desktop/Kubernetes_Theory/6.Kubernetes-Services$ kubectl get rs
NAME                           DESIRED   CURRENT   READY   AGE
python-sample-app-5f95f8b87d   2         2         2       132m

```

&nbsp;

### **Kubernetes Service**

* * *

**7.Go inside cluster**

```
6.Kubernetes-Services$ minikube ssh
Linux minikube 6.12.54-linuxkit #1 SMP PREEMPT_DYNAMIC Tue Nov  4 21:39:03 UTC 2025 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
docker@minikube:~$ 
```

```
docker@minikube:~$ curl -L http://10.244.0.4:8000/demo
```

```
<header>
  <h2>Free DevOps Course By Abhishek</h2>
</header>

<section>
  <nav>
    <ul>
      <li><a href="www.youtube.com/@AbhishekVeeramalla">YouTube</a></li>
      <li><a href="www.linkedin.com/in/abhishek-veeramalla-77b33996/">LinkedIn</a></li>
      <li><a href="https://telegram.me/abhishekveeramalla">Telegram</a></li>
    </ul>
  </nav>
  
  <article>
    <h1>Agenda</h1>
    <p>Learn DevOps with strong foundational knowledge and practical understanding</p>
    <p>Please Share the Channel with your friends and colleagues</p>
  </article>
</section>

<footer>
  <p>@AbhishekVeeramalla</p>
</footer>

</body>
</html>

```

**we cannot access pods from outside in cluster**

```bash
6.Kubernetes-Services$ curl -L http://10.244.0.4:8000/demo

curl: (7) Failed to connect to 10.244.0.4 port 8000 after 1195 ms: Couldn't connect to server

```

<span style="color: rgb(224, 62, 45);">by-default pods have a cluster network attached to it.we have to logging into cluster and accuses it.form outside we cannot.</span>

**<span style="color: rgb(255, 255, 255);">8.Create service</span>**

service.yml

```
apiVersion: v1
kind: Service
metadata:
  name: python-django-app-service
spec:
  type: NodePort
  selector:
    app: python-sample-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

```
6.Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP        6h30m
python-django-app-service   NodePort    10.101.216.215   <none>        80:30007/TCP   43s

```

```
6.Kubernetes-Services$ minikube ip
192.168.49.2

```