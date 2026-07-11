
&nbsp;

**Build Docker image**

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
docker build -t ravi943/python-sample-app-demo:v1 .
```

- ****`8000`**** is the port your Python app is listening on. Docker and Kubernetes just use that number to know where to send the traffic.
- The port `8000` belongs entirely to your Python/Django application. **It is neither a Kubernetes port nor a Docker port. It is an Application port.**
- ****`0.0.0.0`****: This is a special network address. It tells the app, *"Don't just listen for traffic coming from inside the computer. Listen for traffic coming from ANYWHERE (outside networks, Docker, Kubernetes, etc.)."* If you don't put `0.0.0.0`, the app might block Docker and Kubernetes from talking to it!

&nbsp;

```
Kubernetes-Services$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   29h   v1.35.1

Kubernetes-Services$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS      AGE   IP           NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-bj6nh   1/1     Running   1 (25h ago)   29h   10.244.0.7   minikube   <none>           <none>
python-sample-app-5f95f8b87d-fqvsx   1/1     Running   1 (25h ago)   29h   10.244.0.8   minikube   <none>           <none>

Kubernetes-Services$ kubectl get all
NAME                                     READY   STATUS    RESTARTS      AGE
pod/python-sample-app-5f95f8b87d-bj6nh   1/1     Running   1 (25h ago)   29h
pod/python-sample-app-5f95f8b87d-fqvsx   1/1     Running   1 (25h ago)   29h

NAME                                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes                  ClusterIP   10.96.0.1     <none>        443/TCP        29h
service/python-django-app-service   NodePort    10.99.32.95   <none>        80:30007/TCP   26h

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/python-sample-app   2/2     2            2           29h

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/python-sample-app-5f95f8b87d   2         2         2       29h

Kubernetes-Services$ kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
python-sample-app   2/2     2            2           29h

Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes                  ClusterIP   10.96.0.1     <none>        443/TCP        29h
python-django-app-service   NodePort    10.99.32.95   <none>        80:30007/TCP   26h
```

&nbsp;

## **Kubernetes Service**

**service.yml file**

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

**`selector: app: python-sample-app`**: THIS IS THE MAGIC CONNECTION! The Service looks around the cluster and says, "I need to find all Pods that have the nametag `app: python-sample-app`." Because your Deployment gave its Pods that exact nametag, the Service automatically finds them and connects to them!

**deployment.yml  <span style="color: rgb(255, 255, 255);">(The Blueprint for your Pods)</span>**

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

```
6.Kubernetes-Services$ kubectl apply -f service.yml
service/python-django-app-service configured

```

```bash
Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes                  ClusterIP   10.96.0.1     <none>        443/TCP        19h
python-django-app-service   NodePort    10.99.32.95   <none>        80:30007/TCP   16h
```

```
Kubernetes-Services$ minikube ip
192.168.49.2
```

&nbsp;

* * *

<span style="color: rgb(45, 194, 107);">**Notes**</span>

**<span style="color: rgb(255, 255, 255);">1\. ClusterIP (Default Service Type)</span>**

<span style="color: rgb(255, 255, 255);">ClusterIP is the default service type. It gives your application a permanent internal IP address so that other Pods inside the cluster can talk to it, but it keeps the application hidden from the outside internet</span>

<span style="color: rgb(255, 255, 255);">When to use it:</span>

- <span style="color: rgb(255, 255, 255);">When you need Pods to talk to each other inside the cluster.</span>
- <span style="color: rgb(255, 255, 255);">Example: Your frontend Pod needs to talk to your backend database Pod.</span>
- <span style="color: rgb(255, 255, 255);">NOT accessible from outside the cluster (no internet access)</span>

<span style="color: rgb(255, 255, 255);">**<span>2\. NodePort</span>**</span>

<span style="color: rgb(255, 255, 255);">NodePort opens a specific, static port directly on the **Node (the server)**. This creates a direct path for external traffic to enter the server and reach the application inside.</span>

<span style="color: rgb(255, 255, 255);">**How it works:**</span>

- <span style="color: rgb(255, 255, 255);">Traffic flows: **Internet → Node IP : NodePort → ClusterIP → Pod**</span>
- <span style="color: rgb(255, 255, 255);">In your case: `http://192.168.49.2:30007` → Minikube Node → Service (10.99.32.95:80) → Your Python Pod (port 8000)</span>

<span style="color: rgb(255, 255, 255);">**When to use it:**</span>

- <span style="color: rgb(255, 255, 255);">For **development and testing** (like Minikube).</span>
- <span style="color: rgb(255, 255, 255);">When you need simple external access without complex setup.</span>
- <span style="color: rgb(255, 255, 255);">**NOT recommended for production** (limited load balancing, security concerns).</span>
- <span style="color: rgb(255, 255, 255);">\***Static port**  simply means **fixed** or **permanent**.If it weren't "static," Kubernetes might give your app port 30007 today, but if the server restarted tomorrow, it might randomly give it port 30008.</span>

&nbsp;

<span style="color: rgb(255, 255, 255);">*Why does a NodePort service have a Cluster-IP?*</span>

<span style="color: rgb(255, 255, 255);">Because NodePort is built on top of ClusterIP. The Cluster-IP is the internal engine that actually routes the traffic to the correct Pods. The NodePort just acts as the external front door. When external traffic hits the NodePort, it is immediately passed to the Cluster-IP to finish the routing to the Pods. Therefore, every NodePort service must have an internal Cluster-IP address to function</span>

<span style="color: rgb(255, 255, 255);">**Better Analogy:**</span>

- <span style="color: rgb(255, 255, 255);">**ClusterIP Service** = A basic phone that can only make internal calls (within the cluster)</span>
- <span style="color: rgb(255, 255, 255);">**NodePort Service** = A phone that can make internal calls (ClusterIP) **PLUS** external calls (NodePort)</span>
- <span style="color: rgb(255, 255, 255);">**LoadBalancer Service** = A phone that can make internal calls (ClusterIP) **PLUS** external calls (NodePort) **PLUS** has a receptionist to route calls automatically (LoadBalancer)</span>

<span style="color: rgb(255, 255, 255);">When you create a **NodePort Service**, you are NOT creating a separate ClusterIP service. Instead, you are creating ONE service that has **both capabilities built-in**</span>

&nbsp;

* * *

&nbsp;

```
Kubernetes-Services$ minikube ssh
Linux minikube 6.12.54-linuxkit #1 SMP PREEMPT_DYNAMIC Tue Nov  4 21:39:03 UTC 2025 x86_64
The programs included with the Debian GNU/Linux system are free software;
permitted by applicable law.
```

&nbsp;

**1.Access application from inside cluster (using service)**

```
docker@minikube:~$ curl http://10.99.32.95:80/demo -L
<!DOCTYPE html>
<html lang="en">
<head>
```

We are inside the Minikube Node

ClusterIP service is working perfectly inside the cluster!

- The Service received the request
- The Service used its selector (`app: python-sample-app`) to find your Pods
- The Service forwarded the traffic to port 8000 on one of your Pods
- Your Python app responded with the HTML you see

&nbsp;

**2\. Access application from inside cluster **(using Pod IP directly, bypassing the service)****

```
docker@minikube:~$ curl -L http://10.244.0.7:8000/demo
<!DOCTYPE html>
<html lang="en">
<head>
```

Why do we need Services if Pods have IP addresses?

Pods are temporary and their IP addresses change when they restart. Services provide a permanent, stable IP address that never changes. Services also automatically load balance traffic across multiple Pods, so we don't have to manually track which Pod is running. Without Services, our application would break every time a Pod restarts."

&nbsp;

**3.Access application from outside cluster (*Use NodePort service)***

```
Kubernetes-Services$ minikube service python-django-app-service --url
http://127.0.0.1:37331
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

in browser

```
http://127.0.0.1:37331/demo/
```

or open new terminal

```
Kubernetes-Services$ curl -L http://127.0.0.1:37331/demo
<!DOCTYPE html>
<html lang="en">
<head>
```

&nbsp;

* * *

<span style="color: rgb(45, 194, 107);">***Note***</span>

<span style="color: rgb(255, 255, 255);">`minikube service python-django-app-service --url` -  This command creates a <ins>***network tunnel***</ins> from your computer to the Service inside Minikube.</span>

<span style="color: rgb(255, 255, 255);">**Breaking it down:**</span>

- <span style="color: rgb(255, 255, 255);">`minikube service` = "I want to access a Service"</span>
    
- <span style="color: rgb(255, 255, 255);">`python-django-app-service` = "This is the name of the Service I want to access"</span>
    
- <span style="color: rgb(255, 255, 255);">`--url` = "Just show me the URL, don't open the browser automatically"</span>
    
- It creates a temporary bridge (tunnel) from your localhost (`127.0.0.1`) to the Service
    
- It gives you a URL like `http://127.0.0.1:37331` that you can use to access the app
    

&nbsp;

**On macOS**

```bash
┌─────────────────────────────────────┐
│         macOS Host Machine          │
│                                     │
│  ┌───────────────────────────────┐  │
│  │   Docker Desktop (VM)         │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │   Minikube Container    │  │  │
│  │  │   IP: 192.168.49.2      │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
│                                     │
│  Network route: Direct access ✓    │
└─────────────────────────────────────┘
```

- Docker Desktop on macOS runs inside a **virtual machine (VM)**
- Minikube creates a network interface that's **directly accessible** from the macOS host
- The IP `192.168.49.2` is routable from your Mac

&nbsp;

**On Ubuntu**

```
┌─────────────────────────────────────┐
│       Ubuntu Host Machine           │
│                                     │
│  ┌───────────────────────────────┐  │
│  │   Docker (Native - No VM)     │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │   Minikube Container    │  │  │
│  │  │   IP: 192.168.49.2      │  │  │
│  │  └─────────────────────────┘  │  │
│  │                               │  │
│  │   Isolated Docker Network     │  │
│  └───────────────────────────────┘  │
│                                     │
│  Network route: NOT accessible ✗   │
└─────────────────────────────────────┘
```

- Docker on Linux runs **natively** (no VM)
- Minikube creates a **Docker bridge network**
- This network is **isolated** from your Ubuntu host

&nbsp;

```
Kubernetes-Services$ minikube service python-django-app-service --url
http://127.0.0.1:37331
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

| Part | What is it? | Does it change? | Who creates it? |
| --- | --- | --- | --- |
| **`127.0.0.1`** | Your computer's localhost address | ❌ Never changes | Your operating system (Ubuntu) |
| **`37331` / `42593`** | A random port for the tunnel | ✅ Changes every time | Minikube (picks a free port) |

* * *

&nbsp;

&nbsp;

### Load Balancing

* * *

<span style="color: rgb(45, 194, 107);">**Note**</span>

**Install Kubeshark in laptop**

It's a network observability tool that provides real-time visibility into all the network traffic flowing inside your Kubernetes cluster

- **See all API traffic** - It captures and monitors all traffic flowing in, out, and across your containers, pods, and nodes
    
- **Debug network issues** - When your Services or Pods aren't communicating properly, Kubeshark shows you exactly what's happening
    
- **Protocol-level visibility** - You can see HTTP requests, responses, headers, and payloads in real-time
    
    kubezilla.io
    
- **Learning tool** - As a beginner, it helps you visualize how Kubernetes networking actually works
    

after intsall kubeshark on out laptop

```bash
ravindu@Ravilinux:~$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS       AGE    IP            NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-bj6nh   1/1     Running   3 (109m ago)   2d4h   10.244.0.19   minikube   <none>           <none>
python-sample-app-5f95f8b87d-fqvsx   1/1     Running   3 (109m ago)   2d4h   10.244.0.20   minikube   <none>           <none>

ravindu@Ravilinux:~$ kubectl get svc
NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes                  ClusterIP   10.96.0.1     <none>        443/TCP        2d4h
python-django-app-service   NodePort    10.99.32.95   <none>        80:30007/TCP   2d2h

ravindu@Ravilinux:~$ kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
python-sample-app   2/2     2            2           2d4h

ravindu@Ravilinux:~$ minikube ip
192.168.49.2
```

&nbsp;

1.  **Start Kubeshark (monitoring tool) (to monitor network traffic):**

**Terminal 1**

```bash
Kubernetes-Services$ kubeshark tap
2026-07-11T20:05:55+05:30 INF versionCheck.go:23 > Checking for a newer version...
2026-07-11T20:05:55+05:30 INF tapRunner.go:49 > Using Docker: registry=docker.io/kubeshark tag=
2026-07-11T20:05:55+05:30 INF tapRunner.go:53 > Kubeshark will store the traffic up to a limit (per node). Oldest TCP/UDP streams will be removed once the limit is reached. limit=10Gi
2026-07-11T20:05:55+05:30 INF common.go:69 > Using kubeconfig: path=/home/ravindu/.kube/config
2026-07-11T20:05:55+05:30 INF tapRunner.go:69 > Telemetry enabled=true notice="Telemetry can be disabled by setting the flag: --telemetry-enabled=false"
2026-07-11T20:05:55+05:30 INF tapRunner.go:71 > Targeting pods in: namespaces=["default","kube-node-lease","kube-public","kube-system"]
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: python-sample-app-5f95f8b87d-bj6nh
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: python-sample-app-5f95f8b87d-fqvsx
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: coredns-7d764666f9-l7fph
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: coredns-7d764666f9-s6cjl
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: etcd-minikube
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: kube-apiserver-minikube
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: kube-controller-manager-minikube
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: kube-proxy-btkqx
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: kube-scheduler-minikube
2026-07-11T20:05:55+05:30 INF tapRunner.go:138 > Targeted pod: storage-provisioner
2026-07-11T20:05:55+05:30 INF tapRunner.go:81 > Waiting for the creation of Kubeshark resources...
2026-07-11T20:05:56+05:30 WRN versionCheck.go:48 > There is a new release! v53.2.5 -> v53.3.0 Please upgrade to the latest release, as new releases are not always backward compatible. Run: command="sh <(curl -Ls https://kubeshark.com/install)"
2026-07-11T20:05:57+05:30 INF helm.go:131 > Downloading Helm chart: repo-path=/home/ravindu/.cache/helm/repository url=https://github.com/kubeshark/kubeshark.github.io/releases/download/kubeshark-53.3.0/kubeshark-53.3.0.tgz
2026-07-11T20:05:58+05:30 INF helm.go:150 > Installing using Helm: kube-version=">= 1.16.0-0" release=kubeshark source=["https://github.com/kubeshark/kubeshark/tree/master/helm-chart"] version=53.3.0
2026-07-11T20:05:59+05:30 INF helm.go:61 > creating 21 resource(s)
2026-07-11T20:05:59+05:30 INF tapRunner.go:98 > Installed the Helm release: kubeshark
2026-07-11T20:05:59+05:30 INF tapRunner.go:275 > Added: pod=kubeshark-front
2026-07-11T20:05:59+05:30 INF tapRunner.go:179 > Added: pod=kubeshark-hub
2026-07-11T20:07:59+05:30 ERR tapRunner.go:245 > Pod was not ready in time. pod=kubeshark-hub
2026-07-11T20:07:59+05:30 ERR tapRunner.go:339 > Pod was not ready in time. pod=kubeshark-front
2026-07-11T20:07:59+05:30 WRN tapRunner.go:118 > To re-establish a proxy/port-forward, run: command="kubeshark proxy"
```

**2. Open browser**

Go to: **http://127.0.0.1:8899**

**3\. Terminal 2**

```
ravindu@Ravilinux:~$ minikube service python-django-app-service --url
http://127.0.0.1:34829
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

**4\. Terminal 3**

```
ravindu@Ravilinux:~$ curl -L http://127.0.0.1:34829
```

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

**![t1.png](/6.Kubernetes-Services/img/t1.png)**

&nbsp;

&nbsp;

**![t2.png](/6.Kubernetes-Services/img/t2.png)**

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;