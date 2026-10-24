## 1\. Activity 1

### 1.1 Start Kubernetes cluster using minikube

**This created the entire Kubernetes cluster inside one Docker container on your laptop. **So control plane + worker node run inside ONE Docker contain**er. Cluster name = **minikube.(default name)****

```bash
Ravilinux:~$ minikube start
😄  minikube v1.38.1 on Ubuntu 24.04
✨  Automatically selected the docker driver
💨  For improved Docker performance, enable the overlay Linux kernel module using 'modprobe overlay'
❗  Starting v1.39.0, minikube will default to "containerd" container runtime. See #21973 for more info.

⛔  Requested memory allocation (1870MB) is less than the recommended minimum 1900MB. Deployments may fail.

📌  Using Docker driver with root privileges
❗  For an improved experience it's recommended to use Docker Engine instead of Docker Desktop.
Docker Engine installation instructions: https://docs.docker.com/engine/install/#server
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.50 ...
🔥  Creating docker container (CPUs=2, Memory=1870MB) ...
🐳  Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default




```

<img src="/4.Minikube-Activity-1/Images/minikube.png" alt="6d298b3fcd1187c7662575090979bbc3.png" width="660" height="601" class="jop-noMdConv">

### 1.2 Checking nodes

```
Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   72m   v1.35.1
```

<span style="color: rgb(22, 145, 121);">**This command shows all the nodes in the cluster. A node is a machine — physical, virtual, or a container — where Kubernetes runs workloads.**</span>

Name    =  minikube  - Your node name  -  Only 1 node exists (single-node cluster)

**<span style="color: rgb(22, 145, 121);">Control plane node ALSO acts as worker</span>**  
            **<span style="color: rgb(22, 145, 121);">→ Same node runs both roles</span>**  
            **<span style="color: rgb(22, 145, 121);">→ This is called "single-node cluster"</span>**

&nbsp;

- **Process of the docker**

```
DOCKER WORLD
│
├── You
│     → docker run nginx
│
├── Docker CLI
│     → Sends command to daemon
│
├── Docker Engine (dockerd)
│     → Extra layer! Manages images, volumes, networks
│
├── containerd
│     → Manages container lifecycle
│
├── runc
│     → Low-level container runner
│
└── container ✅
      → nginx is live
```

&nbsp;

- **Process of the kubernetes**

```
YOU
 ↓ kubectl command
 ↓
CONTROL PLANE
│
├── kube-api-server    ← receives your request
│         ↓
├── etcd               ← saves desired state
│         ↓
├── kube-controller-manager  ← notices new request
│         ↓                    decides WHAT to do
├── kube-scheduler     ← decides WHERE to run
│         ↓
│   tells API server → saves in etcd
│
WORKER NODE (chosen by scheduler)
│
├── kubelet            ← gets instruction from API server
│         ↓
├── CRI                ← creates the container
│         ↓
└── Pod running! ✅
│
├── kube-proxy         ← makes sure network traffic
                         reaches the pod correctly!
```

&nbsp;

- **Important thing to know:**

```
MINIKUBE vs PRODUCTION
│
├── Minikube (your laptop)
│     → 1 control plane max
│     → Multiple workers possible
│     → For LEARNING only
│
└── Production Kubernetes
      → 3 or 5 control planes (for high availability)
      → Many worker nodes (10, 50, 100+)
      → Real clusters never use single control plane
```

&nbsp;

### 1.3 Create YAML file and run.

```
Ravilinux:~$ vi pod.yml
```

**Add script**

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

every Kubernetes object must have — `apiVersion`, `kind`, `metadata`, and `spec`. Think of it like a form you fill out to tell Kubernetes what you want to create.

- `apiVersion: v1` — which Kubernetes API version to use. Pods always use v1.
- `kind: Pod` — what type of object you want to create.
- `metadata: name: nginx` — just the name of your pod. You will use this name later in kubectl commands.
- `spec` — the actual content. Here you define what containers run inside the pod, which image to use (`nginx:1.14.2` from Docker Hub), and which port the container listens on (port 80).

One thing worth knowing about `containerPort: 80` — this is just documentation. It does NOT actually expose the port to the outside world. It just tells Kubernetes "this app uses port 80." To actually access it from outside, you need a **Service** object, which comes later.

&nbsp;

### 1.4 Create a Pod in the Kubernetes cluster

```
Ravilinux:~$kubectl create -f pod.yml
```

&nbsp;

- **`f` flag tells kubectl to read from a file. The `pod.yml` file contained the Pod definition — the image, name, and port.**
- **This command sent the YAML to the API server, which then scheduled and created the Pod on the node.**

&nbsp;

### **1.5 Checking pods and nodes.**

```
Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   4h25m   v1.35.1

Ravilinux:~$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          6m9s

Ravilinux:~$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          6m25s   10.244.0.3   minikube   <none>           <none>




```

docker run -d  --name nginx -p 80:80  nginx:1.4.2

`kubectl get nodes` — <span style="color: rgb(22, 145, 121);">"**<span style="color: rgb(22, 145, 121);">T</span>his command shows all the nodes in the cluster. A node is a machine — physical, virtual, or a container — where Kubernetes runs workloads."**</span>

`kubectl get pods` —  <span style="color: rgb(22, 145, 121);">**"This command shows all the pods running in the current namespace. A pod is the smallest deployable unit in Kubernetes — it contains one or more containers."**</span>

&nbsp;

**`kubectl get pods -o wide` extra columns:**

- `IP = 10.244.0.3` — this is your pod's private IP address inside the cluster. This IP only works INSIDE the cluster, not from your laptop browser.
- `NODE = minikube` — this pod is running on the minikube node (makes sense, only one node exists).

&nbsp;

### 1.6 Go inside the cluster

```bash
ravindu@Ravilinux:~$ minikube ssh
Linux minikube 6.12.54-linuxkit #1 SMP PREEMPT_DYNAMIC Tue Nov  4 21:39:03 UTC 2025 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.


docker@minikube:~$ curl 10.244.0.3
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

```

&nbsp;

**`minikube ssh`** — this command SSH's you inside the minikube container (your cluster). So now you are literally inside the Kubernetes cluster, not your laptop terminal. That is why the username changed from `ravindu@Ravilinux` to `docker@minikube`.

**`curl 10.244.0.3`** — since you are now inside the cluster, you can reach the pod's private IP directly. And it returned the nginx welcome HTML page — meaning your nginx container is working perfectly!

This is a very important concept to understand:

**OUTSIDE cluster (your laptop) → Cannot reach 10.244.0.3 ❌**

&nbsp;**INSIDE cluster (minikube ssh) → Can reach 10.244.0.3 ✅**

This is exactly why Kubernetes **Services** exist — to expose pods to the outside world. That is coming next! 💪

&nbsp;

### 1.7 To get detailed information about a specific pod

```bash
Ravilinux:~$ kubectl describe pod nginx

Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sun, 05 Apr 2026 23:32:41 +0530
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  nginx:
    Container ID:   docker://e9a56e40b5a1532a0fa8bf809806c22677c38cc882e8f12d22211b68c2f849a6
    Image:          nginx:1.14.2
    Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 05 Apr 2026 23:32:42 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zwdjm (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-zwdjm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m1s   default-scheduler  Successfully assigned default/nginx to minikube
  Normal  Pulled     3m     kubelet            spec.containers{nginx}: Container image "nginx:1.14.2" already present on machine and can be accessed by the pod
  Normal  Created    3m     kubelet            spec.containers{nginx}: Container created
  Normal  Started    2m59s  kubelet            spec.containers{nginx}: Container started

```

&nbsp;

### 1.8 Delete pods 

```
Ravilinux:~$ kubectl delete pod nginx
pod "nginx" deleted from default namespace

Ravilinux:~$ kubectl get pods
No resources found in default namespace.

Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   9h    v1.35.1

```

&nbsp;

### 1.9 Summery of command today

&nbsp;

1\. `Ravilinux:~$ minikube start`                       **- Start Kubernetes cluster using minikube**

2\. `kubectl get nodes`                                             **\-  This command shows all the nodes in the cluster**

3\. `Ravilinux:~$ vi pod.yml`                                **- Create yml file**

4\. `Ravilinux:~$kubectl create -f pod.yml`  **\- Create a Pod in the Kubernetes cluster**

&nbsp;

```
Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   4h25m   v1.35.1

Ravilinux:~$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          6m9s

Ravilinux:~$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          6m25s   10.244.0.3   minikube   <none>           <none>
```

&nbsp;

5.`Ravilinux:~$ minikube ssh`                                     -  **Go inside the cluster**

6\. `docker@minikube:~$ curl 10.244.0.3`              

7\. `Ravilinux:~$ kubectl describe pod nginx`     **- To get detailed information about a specific pod**

8\. `Ravilinux:~$ kubectl delete pod nginx`         -  Delete pods

&nbsp;