
### 1\. Start Kubernetes cluster using minikub

```bash
Ravilinux:~$ minikube start

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


Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   30s   v1.35.1

Ravilinux:~$ kubectl get pods
No resources found in default namespace.

```

&nbsp;

### **2\. lists all namespaces in the cluster**

```
Ravilinux:~$ kubectl get namespaces

NAME              STATUS   AGE
default           Active   5m50s
kube-node-lease   Active   5m50s
kube-public       Active   5m50s
kube-system       Active   5m50s

```

Namespaces are **logical separations inside a cluster**.

`default` where user applications run,

`kube-system` where Kubernetes internal components run

`kube-public` for publicly accessible information

`kube-node-lease` for node heartbeat signals

&nbsp;

### 3\. Shows all resources in the **default namespace**

```
Ravilinux:~$ kubectl get all

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22m

```

`service/kubernetes` which is the internal Kubernetes API service. No pods are showing because I had not deployed any applications to the default namespace yet.

&nbsp;

### **4\. Shows all resources across all namespaces in the cluster**

```
Ravilinux:~$ kubectl get all -A

NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE
kube-system   pod/coredns-7d764666f9-7cf96           1/1     Running   0             24m
kube-system   pod/coredns-7d764666f9-jmqhl           1/1     Running   0             24m
kube-system   pod/etcd-minikube                      1/1     Running   0             24m
kube-system   pod/kube-apiserver-minikube            1/1     Running   0             24m
kube-system   pod/kube-controller-manager-minikube   1/1     Running   0             24m
kube-system   pod/kube-proxy-stcqv                   1/1     Running   0             24m
kube-system   pod/kube-scheduler-minikube            1/1     Running   0             24m
kube-system   pod/storage-provisioner                1/1     Running   1 (23m ago)   24m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  24m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   24m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   24m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           24m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-7d764666f9   2         2         2       24m


```

Kubernetes system components running as pods inside `kube-system` namespace — things like `kube-apiserver`, `kube-scheduler`, `kube-controller-manager`, `etcd`, `kube-proxy` and `coredns`. This proves that Kubernetes runs its own internal components as pods inside the cluster."

&nbsp;

- <span style="color: rgb(45, 194, 107);">**Confusion 1 — Control plane components showing as pods?**</span>

<span style="color: rgb(45, 194, 107);">Control plane components are NOT pods . they are separate processes. But in Minikube specifically, Kubernetes runs those control plane components AS pods inside `kube-system` namespace. That is why you see them in `kubectl get all -A`.</span>

```
Real (production)
│
├── Control plane components = separate processes
│     → running directly on the master server
│     → NOT pods
│
└── Worker nodes = kubelet, kube-proxy, pods

MINIKUBE (your laptop)
│
├── Control plane components = run AS pods
│     → kube-apiserver-minikube
│     → kube-scheduler-minikube
│     → kube-controller-manager-minikube
│     → etcd-minikube
│     → all inside kube-system namespace
│
└── This is called "static pods"
      → Minikube wraps them as pods for simplicity
```

<span style="color: rgb(45, 194, 107);">Minikube just packages everything as pods to make it easier to manage on a single machine. In real production clusters, control plane components run as system processes directly on the master server, not as pods.</span>

&nbsp;

- <span style="color: rgb(45, 194, 107);">**Confusion 2 — `service/kubernetes` vs `kube-apiserver` pod, are they the same?**</span>

```
Ravilinux:~$ kubectl get all

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22m

```

<span style="color: rgb(45, 194, 107);">They are related but NOT the same thing. Think of it this way:</span>

<span style="color: rgb(45, 194, 107);">"`service/kubernetes` is a Kubernetes Service object that provides a stable IP address (`10.96.0.1`) to reach the **kube-apiserver** from **inside the cluster**. The actual kube-apiserver runs as a pod in `kube-system` namespace, but pods can get new IP addresses when they restart. The Service solves this by giving a permanent fixed IP that never changes . so other pods inside the cluster always know how to reach the API server reliably."</span>

<span style="color: rgb(45, 194, 107);">**Now the problem it solves:**</span>

<span style="color: rgb(45, 194, 107);">Pods are temporary. When a pod restarts it can get a **new IP address**. So if something was using the old IP  connection breaks. This is the problem Service solves.</span>

```
WITHOUT Service
│
├── kube-apiserver pod starts  → IP = 10.244.0.5
├── other pods talk to 10.244.0.5  ✅
├── kube-apiserver pod restarts → IP changes to 10.244.0.8 !!
└── other pods still try 10.244.0.5 → FAIL ❌

WITH Service (service/kubernetes = 10.96.0.1)
│
├── kube-apiserver pod starts  → IP = 10.244.0.5
├── other pods talk to 10.96.0.1 (Service) ✅
├── kube-apiserver pod restarts → IP changes to 10.244.0.8
├── Service automatically updates → now points to 10.244.0.8
└── other pods still talk to 10.96.0.1 → STILL WORKS ✅
```

So in your output:

`pod/kube-apiserver-minikube`   →   actual kube-apiserver. IP can change when pod restarts

`service/kubernetes 10.96.0.1` → fixed forever(IP). Other pods inside the cluster always talk to kube-apiserver through this stable IP.Even if kube-apiserver pod restarts and gets a new IP, the service automatically routes to the new IP. So communication never breaks.

&nbsp;

### 5\. Create Pod.yml file

```
Ravilinux:~$ ls
'3. Docker Install.pdf'   Desktop   Dockerfile   Documents   Downloads   JoplinBackup   kubectl   kubectl.sha256   Music   opt   Pictures   pod.yml   Public   snap   Templates   Videos

ravindu@Ravilinux:~$ vim pod.yml

```

```
Ravilinux:~$ vim pod.yml
```

- **pod.yml file**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80


```

&nbsp;

### 6\. Create a pod in the kubernetes cluster

```
Ravilinux:~$ kubectl apply -f pod.yml
pod/nginx created

Ravilinux:~$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          4m14s

Ravilinux:~$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          5m47s   10.244.0.5   minikube   <none>           <none>


```

**7\. Loging to minikube cluster**

```
Ravilinux:~$ minikube ssh
Linux minikube 6.12.54-linuxkit #1 SMP PREEMPT_DYNAMIC Tue Nov  4 21:39:03 UTC 2025 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

docker@minikube:~$ 

```

**Application running**

&nbsp;

```
docker@minikube:~$ curl 10.244.0.5

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
docker@minikube:~$ 

```

&nbsp;

### **8\. Some reason pod is delete**

(Now application is not reachable. now cant access nginx pods from outside worlds)

```
Ravilinux:~$ kubectl delete pod nginx
pod "nginx" deleted from default namespace

ravindu@Ravilinux:~$ minikube ssh
Linux minikube 6.12.54-linuxkit #1 SMP PREEMPT_DYNAMIC Tue Nov  4 21:39:03 UTC 2025 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

docker@minikube:~$ curl 10.244.0.5
curl: (7) Failed to connect to 10.244.0.5 port 80 after 14282 ms: Couldn't connect to server
docker@minikube:~$ exit 
logout


```

&nbsp;

<span style="color: rgb(224, 62, 45);">**kuberetes have auto healing, auto scaling option. but now application gone.same thing happend docker also. but what is the benefit of kuberntes.?**</span>

<span style="color: rgb(45, 194, 107);">kubernetes support all of that but we have to create correct resource. in here   we have created a pod instead we have to create deployment.</span>

**<span style="color: rgb(255, 255, 255);">9\. Create deployments.yml file (part 1- replicas: 1)</span>**

```
Ravilinux:~$ vim deployment.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

&nbsp;

### **10\. Create Deployment**

```
Ravilinux:~$ kubectl apply -f deployment.yml
deployment.apps/nginx-deployment created


```

```
ravindu@Ravilinux:~$ kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           4m4s

ravindu@Ravilinux:~$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-77bc6bd484   1         1         1       9m3s

ravindu@Ravilinux:~$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-s8bzd   1/1     Running   0          4m9s



```

Deploy > ReplicaSet > Pods

&nbsp;

### 12\. Show what is happening pod in live\*\*\*\*(before delete pods)

```
Ravilinux:~$ kubectl get pods -w 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-s8bzd   1/1     Running   0          27m



```

**12.1 Delete pod**

```
Ravilinux:~$ kubectl delete pod nginx-deployment-77bc6bd484-s8bzd
pod "nginx-deployment-77bc6bd484-s8bzd" deleted from default namespace

```

&nbsp;

**12.2 Live pod status**

Even after delete pod, automatically create new pods.pod is always running state automatically.

```
Ravilinux:~$ kubectl get pods -w 

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-s8bzd   1/1     Running   0          27m
nginx-deployment-77bc6bd484-s8bzd   1/1     Terminating   0          29m
nginx-deployment-77bc6bd484-s8bzd   1/1     Terminating   0          29m
nginx-deployment-77bc6bd484-bd8lb   0/1     Pending       0          0s
nginx-deployment-77bc6bd484-bd8lb   0/1     Pending       0          0s
nginx-deployment-77bc6bd484-bd8lb   0/1     ContainerCreating   0          0s
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-bd8lb   1/1     Running             0          2s
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m


```

```
Ravilinux:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-bd8lb   1/1     Running   0          6m50s
```

&nbsp;

### 13\. Increase replicaSet (increase replicas to 3 )

```
Ravilinux:~$ vim deployment.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```

&nbsp;

```
Ravilinux:~$ kubectl apply -f deployment.yml 
deployment.apps/nginx-deployment configured

Ravilinux:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-6lhkf   1/1     Running   0          33s
nginx-deployment-77bc6bd484-bd8lb   1/1     Running   0          37m
nginx-deployment-77bc6bd484-pgpgd   1/1     Running   0          33s

Ravilinux:~$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-77bc6bd484-6lhkf   1/1     Running   0          2m5s   10.244.0.9   minikube   <none>           <none>
nginx-deployment-77bc6bd484-bd8lb   1/1     Running   0          39m    10.244.0.7   minikube   <none>           <none>
nginx-deployment-77bc6bd484-pgpgd   1/1     Running   0          2m5s   10.244.0.8   minikube   <none>           <none>
 
```

&nbsp;

```
Ravilinux:~$ kubectl get pods -w
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-6lhkf   1/1     Running   0          6m59s
nginx-deployment-77bc6bd484-bd8lb   1/1     Running   0          44m
nginx-deployment-77bc6bd484-pgpgd   1/1     Running   0          6m59s

```

&nbsp;

\*\*\*

```
ravindu@Ravilinux:~$ kubectl get pods -w 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-s8bzd   1/1     Running   0          27m
nginx-deployment-77bc6bd484-s8bzd   1/1     Terminating   0          29m
nginx-deployment-77bc6bd484-s8bzd   1/1     Terminating   0          29m
nginx-deployment-77bc6bd484-bd8lb   0/1     Pending       0          0s
nginx-deployment-77bc6bd484-bd8lb   0/1     Pending       0          0s
nginx-deployment-77bc6bd484-bd8lb   0/1     ContainerCreating   0          0s
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-bd8lb   1/1     Running             0          2s
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-s8bzd   0/1     Completed           0          29m
nginx-deployment-77bc6bd484-pgpgd   0/1     Pending             0          0s
nginx-deployment-77bc6bd484-6lhkf   0/1     Pending             0          0s
nginx-deployment-77bc6bd484-pgpgd   0/1     Pending             0          0s
nginx-deployment-77bc6bd484-6lhkf   0/1     Pending             0          0s
nginx-deployment-77bc6bd484-pgpgd   0/1     ContainerCreating   0          0s
nginx-deployment-77bc6bd484-6lhkf   0/1     ContainerCreating   0          0s
nginx-deployment-77bc6bd484-6lhkf   1/1     Running             0          2s
nginx-deployment-77bc6bd484-pgpgd   1/1     Running             0          2s

```

- \*\*\*\*

```
Ravilinux:~$ kubectl delete pods nginx-deployment-77bc6bd484-bd8lb 
pod "nginx-deployment-77bc6bd484-bd8lb" deleted from default namespace

```

&nbsp;

```
Ravilinux:~$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-77bc6bd484-6lhkf   1/1     Running   0          17m   10.244.0.9    minikube   <none>           <none>
nginx-deployment-77bc6bd484-n29hh   1/1     Running   0          96s   10.244.0.10   minikube   <none>           <none>
nginx-deployment-77bc6bd484-pgpgd   1/1     Running   0          17m   10.244.0.8    minikube   <none>           <none>

```

&nbsp;

```
Ravilinux:~$ kubectl get pods -w
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bc6bd484-6lhkf   1/1     Running   0          6m59s
nginx-deployment-77bc6bd484-bd8lb   1/1     Running   0          44m
nginx-deployment-77bc6bd484-pgpgd   1/1     Running   0          6m59s
nginx-deployment-77bc6bd484-bd8lb   1/1     Terminating   0          52m
nginx-deployment-77bc6bd484-bd8lb   1/1     Terminating   0          52m
nginx-deployment-77bc6bd484-n29hh   0/1     Pending       0          0s
nginx-deployment-77bc6bd484-n29hh   0/1     Pending       0          0s
nginx-deployment-77bc6bd484-n29hh   0/1     ContainerCreating   0          0s
nginx-deployment-77bc6bd484-bd8lb   0/1     Completed           0          52m
nginx-deployment-77bc6bd484-bd8lb   0/1     Completed           0          52m
nginx-deployment-77bc6bd484-bd8lb   0/1     Completed           0          52m
nginx-deployment-77bc6bd484-n29hh   1/1     Running             0          2s



```