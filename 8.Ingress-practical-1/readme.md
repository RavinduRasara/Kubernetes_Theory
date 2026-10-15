## 1. Write docker file and Push Docker image to Docker Hub

### 1.1 HTML File

```
<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
</html>
```

### 1.2 Docker File
```
# Start with lightweight Alpine (Nginx is already installed inside this!)
FROM nginx:alpine

# Set the working directory to Nginx's default web folder
WORKDIR /usr/share/nginx/html

# Copy your HTML file into that folder
COPY index.html .

# Tell Docker that the container listens on port 80 at runtime
EXPOSE 80

# Tell Docker what to run when it starts
# "daemon off;" keeps Nginx running in the foreground so the container doesn't exit
CMD ["nginx", "-g", "daemon off;"]
```
bcz The container stays alive ONLY as long as the main command is running in the foreground. The container will die in less than a second. When you try to open http://localhost:8080 in your browse

```
7.Kubernetes-Ingress$ ls
Dockerfile  img  index.html  ingress.yml  readme.  
```

### 1.3 Build docker image 
```
7.Kubernetes-Ingress$ docker build -t ravi943/harry-app:1.0 .
[+] Building 133.7s (9/9) FINISHED                                                                                                                  docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                0.5s
 => => transferring dockerfile: 713B                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine      
```

## 1.4 Run docker Container in locally 
```
7.Kubernetes-Ingress$ docker run -p 8081:80 ravi943/harry-app:1.0
```
Check in browser : http://localhost:8081/

### 1.5 Push Docker image to docker hub
```
7.Kubernetes-Ingress$ docker login
Authenticating with existing credentials... 
Login Succeeded

7.Kubernetes-Ingress$ docker push ravi943/harry-app:1.0
The push refers to repository [docker.io/ravi943/harry-app]
7c95cc9bf7aa: Pushed 
cfc08d7798ef: Pushed 
2e2ad212a9f4: Pushed 
```

![[Kubernetes-8-dockerhub.png|198]]

---
## 2. Kubernetes Deployment

###  2.1 deployment.yaml
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
        ports:
        - containerPort: 80
```

`spec.template.metadata.labels` (The Pod's Actual Name Tag) ⭐️ THIS IS THE IMPORTANT ONE. **Who cares**: The Service's `selector` looks around the cluster for Pods wearing this exact name tag.
`spec.selector.matchLabels`  This tells the Deployment (the manager): _"Only manage Pods that are wearing this specific name tag._ **Who cares:** Only the Deployment. The Service **does not care** about this.
`metadata.labels` (The Deployment's Name Tag) - This pins a name tag on the **Deployment object itself**.

### 2.2 service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: harry-service
spec:
  selector:
    app: harry-app 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

The **Service `selector`** must **always exactly match** the **Deployment `template.metadata.labels`**.

### 2.3 Ingress.yml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: harry-ingress
spec:
  rules:
  - host: harry.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: harry-service                  
            port:
              number: 80

```
⭐️ spec.rules.backend.service.name : harry-service must match with service.yaml metadata.name
 host: harry.local - Host base routing 
 remove ingress class.
 
- Trying to define the traffic rules for my service.
### 2.4 Start minikube 

```
Ravilinux:~$ minikube start
😄  minikube v1.38.1 on Ubuntu 24.04
✨  Using the docker driver based on existing profile
💨  For improved Docker performance, enable the overlay Linux kernel module using 'modprobe overlay'
```

```
8.Ingress-practical-1$ ls
deployment.yaml  img  service.yaml
```

```
8Ingress-practical-1$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   14m

8Ingress-practical-1$ minikube ip
192.168.49.2

8Ingress-practical-1$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   14m

```

### 2.5 Apply deployment , service yaml file

```
8.Ingress-practical-1$ kubectl apply -f deployment.yaml
deployment.apps/harry-deployment created

8.Ingress-practical-1$ kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
harry-deployment   2/2     2            2           3m35s

8.Ingress-practical-1$ kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
harry-deployment-87d4bb4d6-9mlqg   1/1     Running   0          5m37s   10.244.0.5   minikube   <none>           <none>
harry-deployment-87d4bb4d6-vjwdz   1/1     Running   0          5m37s   10.244.0.4   minikube   <none>           <none>

8.Ingress-practical-1$ kubectl apply -f service.yaml
service/harry-service created

8.Ingress-practical-1$ kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
harry-service   ClusterIP   10.109.44.94   <none>        80/TCP    61s
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   27m
```

### 2.6  Enable the Ingress Controller

```
8Ingress-practical-1$ minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS

```

check Ingress Controller pods are running.
```
8Ingress-practical-1$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-gsr2c        0/1     Completed   0          40m
ingress-nginx-admission-patch-n7mlj         0/1     Completed   0          40m
ingress-nginx-controller-596f8778bc-9tf9g   1/1     Running     0          40m
```
**`-n`** means **"specify a specific namespace"**(for example, `-n ingress-nginx` tells it to look in the `ingress-nginx` namespace.not default)

```
8Ingress-practical-1$ kubectl apply -f ingress.yaml
ingress.networking.k8s.io/harry-ingress created
```

```
8Ingress-practical-1$ kubectl get ing
NAME            CLASS   HOSTS         ADDRESS        PORTS   AGE
harry-ingress   nginx   harry.local   192.168.49.2   80      69s
```

### 2.7 Checking service Endpoint and change /etc/host file 
```
8Ingress-practical-1$ kubectl get endpoints harry-service
aWarning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME            ENDPOINTS                       AGE
harry-service   10.244.0.14:80,10.244.0.15:80   2d8h
```

```
8.Ingress-practical-1$ sudo vim /etc/hosts
```
Add- 
End of section
127.0.0.1 harry.local

The `/etc/hosts` file is a shortcut that skips this ask-a-server step — it's a local lookup table on your own machine.
"whenever I type `harry.local`, go to this IP."

### 2.8 Create minikube tunnel 

```
8.Ingress-practical-1$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress harry-ingress requires privileged ports to be exposed: [80 443]
```


- **Docker Desktop**  always uses an internal VM. minikube's internal network  exists _inside_ Docker Desktop's VM. so it can't directly route to `192.168.49.2`. but Normally on **native Linux**, Docker runs directly on the Linux kernel — no VM needed.

- DNS = "Domain Name System." When you type a name like `google.com` into your browser or `curl`, our  computer doesn't know how to reach a _name_ — it only knows how to reach IP addresses (like `142.250.1.1`). So it does a "DNS lookup": it asks a DNS server "what IP address does this name point to?" and gets an answer back, then connects to that IP.

- By default, our  DNS server is usually your **router** (home Wi-Fi router), which forwards requests to your **ISP's DNS server**.

**1. What is `/etc/hosts` and why add it?**

- `/etc/hosts` is just a text file on your computer that works like a personal phone book — "this name = this IP address." When you type `harry.local` anywhere (browser, curl), your computer checks this file first before trying real DNS.

- `harry.local` isn't a real internet domain, so nothing knows what it means unless you tell your computer yourself. That's why you add a line — you're saying "whenever I type `harry.local`, go to this IP."

- Why `127.0.0.1` specifically (not the minikube IP)? Because `minikube tunnel` works by forwarding traffic to your own machine (`127.0.0.1` = "this same computer") and then internally routing it into the cluster. So once the tunnel is running, the correct address to point at is your own machine, not `192.168.49.2` directly.

**2. Why `192.168.49.2 harry.local` didn't work for you**

- This matches exactly what we found in your earlier Service tutorial session: because you have **Docker Desktop** installed, minikube's network (`192.168.49.x`) is hidden inside Docker Desktop's internal VM. Your host laptop simply cannot reach `192.168.49.2` at all — not with curl, not even with `ping`. That's why it times out.

- `minikube tunnel` exists specifically to fix this — it builds a bridge so traffic sent to `127.0.0.1` on your host gets carried into that hidden network. That's why the fix is: tunnel running + `/etc/hosts` pointing to `127.0.0.1`, not the minikube IP.

### 2.9 Run the application from terminal and browser

```
8.Ingress-practical-1$ curl harry.local

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
</html>
```

http://harry.local/

```
8.Ingress-practical-1$ curl 127.0.0.1 -H 'Host: harry.local'

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
</html>

```

**tunnel starts at your host (`127.0.0.1`) and ends at the ingress controller inside minikube's network, bypassing the unreachable middle layer (Docker Desktop's internal VM boundary).**


                              USER
                               │
                               │ curl harry.local
                               │ (via minikube tunnel → 127.0.0.1)
                               ▼
                    ┌───────────────────────┐
                    │     minikube tunnel     │
                    │  (host ↔ cluster bridge)│
                    └───────────┬───────────┘
                                │
                                ▼
        ┌───────────────────────────────────────────────┐ 
        │                minikube cluster                 │
        │                                                 │
        │    ┌─────────────────────────────┐              │
        │    │      Ingress Controller     │              │
        │    │           (nginx)           │              │
        │    │   matches Host: harry.local │              │
        │    └───────────────┬─────────────┘              │
        │                    │ port.number: 80            │
        │                    ▼                            │
        │    ┌─────────────────────────────┐              │
        │    │         harry-service       │              │
        │    │   ClusterIP  10.109.44.94:80│              │
        │    └───────────────┬─────────────┘              │
        │                    │ targetPort: 80             │
        │                    ▼                            │
        │      ┌───────────────┬───────────────┐          │
        │      │      Pod 1       │      Pod 2            │
        │      │ 10.244.0.24:80   │ 10.244.0.25:80        │
        │      │ containerPort:80 │ containerPort:80      │
        │      └───────────────┴───────────────┘          │
        │                                                 │
        └───────────────────────────────────────────────┘

**![kubernetes-8-ingress.png](/8.Ingress-practical-1/img/kubernetes-8-ingress.png)**     **![kubenetes-8-service.yaml.png](/8.Ingress-practical-1/img/kubenetes-8-service.yaml.png)
                        
```
8.Ingress-practical-1$ kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
harry-service   ClusterIP   10.109.44.94   <none>        80/TCP    61s
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   27m
```

```
8Ingress-practical-1$ kubectl get ing
NAME            CLASS   HOSTS         ADDRESS        PORTS   AGE
harry-ingress   nginx   harry.local   192.168.49.2   80      69s
```


- **`ingress.yaml` → `port.number: 80`**  
   This tells the Ingress "send matching traffic to the Service on port 80." This number must equal the Service's `port` field (not `targetPort`). Ingress doesn't care about pods at all — it only talks to the Service.

- **`service.yaml` → `port: 80`**  
   This is the port the Service itself exposes to anything calling it (Ingress, or other pods inside the cluster). This is the number that shows up in `kubectl get svc` as `80/TCP`.

- **`service.yaml` → `targetPort: 80`**  
   This is the port the Service forwards traffic _to_ on the pod. It must equal the pod's actual listening port.

- **`deployment.yaml` → `containerPort: 80`**  
  This is just documentation of which port your app inside the container is actually listening on (your HTML/app process itself). `targetPort` must match this number, or requests will reach the pod but get refused/dropped since nothing's listening there.



---

## 3.Path base routing


**Application 1**
```
<!DOCTYPE html>

<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
</html>
```

**Dockerfile 1**
```
FROM nginx:alpine
WORKDIR /usr/share/nginx/html/first
COPY index.html .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
**/first

**Application 2**
```
<!DOCTYPE html>
<html>
<body>

<h1> The Lord of the Rings</h1>
<h2>Hi,i am Aragorn</h2>
<h3>Hi,i am Gandalf</h3>
<h4>Hi,i am Bilbo Baggins</h4>
<h5>Hi,i am Frodo Baggins</h5>
<h6>Ha ha ha! I am not Gandalf.I am Sauron</h6>

</body>
</html>
```

**Dockerfile 2**
```
FROM nginx:alpine
WORKDIR /usr/share/nginx/html/second
COPY index.html .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
**/second

### 3.1 create deployment(2), service(2), ingress(1) yaml file

```
8.Ingress-practical-1$ ls
deployment1.yaml  deployment2.yaml  deployment.yaml  img  ingress1.yaml  ingress.yaml  pathbase-ingress.yaml  readme.md  service1.yaml  service2.yaml  service.yaml
```
 
**deployment1.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hogwarts-deployment
  labels:
    app: hogwarts-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hogwarts-app
  template:
    metadata:
      labels:
        app: hogwarts-app
    spec:
      containers:
      - name: hogwarts-container
        image: ravi943/hogwarts:1.0
        ports:
        - containerPort: 80
```

**service1.yaml**

```
apiVersion: v1
kind: Service
metadata:
  name: hogwarts-service
spec:
  selector:
    app: hogwarts-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

**deployment2.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: middle-earth-deployment
  labels:
    app: middle-earth-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: middle-earth-app
  template:
    metadata:
      labels:
        app: middle-earth-app
    spec:
      containers:
      - name: middle-earth-container
        image: ravi943/middle-earth:1.0
        ports:
        - containerPort: 80
```

**service2.yaml**

```
apiVersion: v1
kind: Service
metadata:
  name: middle-earth-service
spec:
  selector:
    app: middle-earth-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

**ingress1.yaml**

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-auth
spec:
  rules:
  - host: hogwarts.middle.earth
    http:
      paths:
      - path: /first
        pathType: Prefix
        backend:
          service:
            name: hogwarts-service
            port:
              number: 80
      - path: /second
        pathType: Prefix
        backend:
          service:
            name: middle-earth-service
            port:
              number: 80
```

```
8.Ingress-practical-1$ sudo vim /etc/hosts
[sudo] password for ravindu: 
```

 End of section
127.0.0.1 hogwarts.middle.earth

### 3.2 Apply deployment, service, ingress yaml file

```
8.Ingress-practical-1$ kubectl apply -f deployment1.yaml -f service1.yaml -f deployment2.yaml -f service2.yaml -f ingress1.yaml
deployment.apps/hogwarts-deployment created
service/hogwarts-service created
deployment.apps/middle-earth-deployment created
service/middle-earth-service created
ingress.networking.k8s.io/ingress-with-auth created
```

### 3.3 Create minikube tunnel

```
8.Ingress-practical-1$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress ingress-with-auth requires privileged ports to be exposed: [80 443]

```

### 3.4 Run application from terminal and browser

```
8.Ingress-practical-1$ curl -L hogwarts.middle.earth/first

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
```

```
8.Ingress-practical-1$ curl -L hogwarts.middle.earth/second

<!DOCTYPE html>
<html>
<body>

<h1> The Lord of the Rings</h1>
<h2>Hi,i am Aragorn</h2>
<h3>Hi,i am Gandalf</h3>
<h4>Hi,i am Bilbo Baggins</h4>
<h5>Hi,i am Frodo Baggins</h5>
<h6>Ha ha ha! I am not Gandalf.I am Sauron</h6>

</body>
```

Browser
```
http://hogwarts.middle.earth/first/
```

```
http://hogwarts.middle.earth/second/
```

### 3.5 Note

To check end point
```
8.Ingress-practical-1$ kubectl get endpoints hogwarts-service 
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME               ENDPOINTS        AGE
hogwarts-service   10.244.0.46:80   40m


8.Ingress-practical-1$ kubectl get endpoints middle-earth-service
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME                   ENDPOINTS        AGE
middle-earth-service   10.244.0.47:80   42m

```


To manually clear minikube's cached copy
```
8.Ingress-practical-1$ minikube image rm ravi943/middle-earth:1.0
```

To list all images minikube's node has cached
```
8.Ingress-practical-1$ minikube image ls

registry.k8s.io/pause:3.10.1
registry.k8s.io/kube-scheduler:v1.35.1
registry.k8s.io/kube-proxy:v1.35.1
registry.k8s.io/kube-controller-manager:v1.35.1
registry.k8s.io/kube-apiserver:v1.35.1
registry.k8s.io/etcd:3.6.6-0
registry.k8s.io/coredns/coredns:v1.13.1
gcr.io/k8s-minikube/storage-provisioner:v5
docker.io/ravi943/hogwarts:1.0
docker.io/ravi943/harry-app:1.0
```

To search for a specific one:
```
8.Ingress-practical-1$ minikube image ls | grep middle-earth
```

```
8.Ingress-practical-1$ kubectl get all
NAME                                           READY   STATUS    RESTARTS      AGE
pod/hogwarts-deployment-6d9f7fd957-7gwm7       1/1     Running   2 (22h ago)   3d21h
pod/middle-earth-deployment-5568c86c74-2lpx7   1/1     Running   2 (22h ago)   3d19h

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/hogwarts-service       ClusterIP   10.108.222.108   <none>        80/TCP    3d21h
service/kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP   12d
service/middle-earth-service   ClusterIP   10.98.183.113    <none>        80/TCP    3d21h

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hogwarts-deployment       1/1     1            1           3d21h
deployment.apps/middle-earth-deployment   1/1     1            1           3d19h

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/hogwarts-deployment-6d9f7fd957       1         1         1       3d21h
replicaset.apps/middle-earth-deployment-5568c86c74   1         1         1       3d19h
```

```
8.Ingress-practical-1$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   12d
ingress-nginx     Active   11d
kube-node-lease   Active   12d
kube-public       Active   12d
kube-system       Active   12d

r8.Ingress-practical-1$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS       AGE
ingress-nginx-admission-create-gsr2c        0/1     Completed   0              11d
ingress-nginx-admission-patch-n7mlj         0/1     Completed   0              11d
ingress-nginx-controller-596f8778bc-9tf9g   1/1     Running     9 (128m ago)   11d
```


This command reaches into the running ingress-controller pod and prints out nginx's actual live configuration file
```
kubectl exec -it -n ingress-nginx deploy/ingress-nginx-controller cat /etc/nginx/nginx.conf
```
kubectl exec - runs a command _inside_ an already-running container
**`-it`** — two combined flags:
- `-i` (interactive) — keeps input open so you could type things
- `-t` (tty) — allocates a terminal, so output looks nice/formatted like a real terminal session
- **`deploy/ingress-nginx-controller`** —  this says "target the pod managed by this Deployment"

### 3.6 wildcard with ingress

A wildcard host uses `*` to match **any subdomain** in that position. So:
```
host: "*.bar.com"
```
This single rule matches `foo.bar.com`, `test.bar.com`, `anything.bar.com` — literally any subdomain of `bar.com` — all routed to the **same** backend service (`http-svc`), without needing a separate rule for each one.
That's the whole point: instead of writing 10 separate `host:` rules for 10 subdomains, one wildcard rule covers all of them.

```
8.Ingress-practical-1$ ls
deployment1.yaml  deployment2.yaml  deployment.yaml  harry-app  img  ingress1.yaml  ingress.yaml  middle-earth  pathbase-ingress.yaml  readme.md  service1.yaml  service2.yaml  service.yaml
```

```
vim wildcard-ingress.yaml
```
Option 1
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wildcard-ingress
spec:
  rules:
  - host: "*.middle.earth"
    http:
      paths:
      - path: /first/
        pathType: Prefix
        backend:
          service:
            name: hogwarts-service
            port:
              number: 80
```

OR

Option 2
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wildcard-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /first/
spec:
  rules:
  - host: "*.middle.earth"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hogwarts-service
            port:
              number: 80
```

- **`nginx.ingress.kubernetes.io/rewrite-target: /first/`** — tells the ingress controller: _"Before forwarding any matched request to the backend service, replace the path with `/first/` instead of whatever the client actually requested."
- Client requests `test1.middle.earth/` → ingress matches the wildcard rule → **before** sending to `hogwarts-service`, it rewrites the path to `/first/` → pod receives a request for `/first/` → finds your `index.html` there → returns it successfully.

```
8.Ingress-practical-1$ kubectl apply -f wildcard-ingress.yaml
ingress.networking.k8s.io/wildcard-ingress created
```

```
8.Ingress-practical-1$ kubectl get ing
NAME                CLASS   HOSTS                   ADDRESS        PORTS   AGE
ingress-with-auth   nginx   hogwarts.middle.earth   192.168.49.2   80      3d23h
wildcard-ingress    nginx   *.middle.earth          192.168.49.2   80      30s

```

For option 1
```
8.Ingress-practical-1$ curl test1.middle.earth/first/

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
```
http://test1.middle.earth/first/
```
8.Ingress-practical-1$ curl test2.middle.earth/first/

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
```


For option 2
```
8.Ingress-practical-1$ curl test1.middle.earth/

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
```

```
8.Ingress-practical-1$ curl test2.middle.earth/

<!DOCTYPE html>
<html>
<body>

<h1>Hi,i am Harry Potter 1</h1>
<h2>Hi,i am Harry Potter 2</h2>
<h3>Hi,i am Harry Potter 3</h3>
<h4>Hi,i am Harry Potter 4</h4>
<h5>Hi,i am Harry Potter 5</h5>
<h6>Ha ha ha! I am not Harry.I am Voldemort</h6>

</body>
```

---
## 3.7 TLS

**TLS (Transport Layer Security)** is the technology that **locks (encrypts) data** while it travels over the internet, so nobody in the middle can read it.

- **SSL** is the old name. TLS is the newer, safer version. People still say "SSL" out of habit, so "SSL passthrough" and "TLS passthrough" mean the same thing.

- **HTTPS = HTTP + TLS.** The padlock in your browser means TLS is working.

### 3.7.1 SSL Passthrough

The load balancer receives the encrypted HTTPS traffic and **forwards it without decrypting it**. 
It acts as a simple relay and never reads the content. The **backend app** does the decryption

```
Client ──encrypted──▶ Load Balancer ──encrypted──▶ App (decrypts here)
                      (just forwards)
```

**Advantages**

- **End-to-end encryption.** Traffic is never decrypted in the middle.
- **Better for strict security or compliance** rules.
- **Supports mutual TLS** (client certificates checked by the app itself).
- **The private key stays on the app**, not on the load balancer.

**Disadvantages**

- **No WAF**, so attacks pass straight to the app.
- **No path-based or session-based routing.**
- **No header changes or traffic inspection.**
- **The app needs to manage its own certificate.**
- **More CPU on the backend pods**, because each pod must decrypt.

### 3.7.2 SSL offloading

**SSL offloading** means the load balancer **decrypts the HTTPS traffic** and then sends **plain HTTP** to the app

```
Client ──HTTPS (encrypted)──▶ Load Balancer ──HTTP (not encrypted)──▶ App
                              (decrypts here)
```

**Advantages**

- It can read the traffic, so a WAF can check and block attacks
- It can see the path, so `/api` and `/shop` go to different services (path-base routing)
- It can read cookies, so the same user can go to the same  (session/cookie routing)
- **Decryption happens once** at the load balancer, so the pods are lighter
- faster. application latency can be reduce.

**Disadvantage of offloading**

- Traffic from the load balancer to the app is **not encrypted**. Inside a trusted cluster this is usually acceptable, but strict security rules may not allow it.
- The load balancer holds the private key, so it becomes an important thing to protect.

### 3.7.3 SSL Bridging

```
Client ──HTTPS──▶ Load Balancer ──HTTPS (new encryption)──▶ App
                  (decrypt → inspect → encrypt again)
```

**SSL bridging** means the load balancer **decrypts** the HTTPS traffic, reads and checks it, and then **encrypts it again** before sending it to the app.

 **Advantages**

- WAF, path routing, and session routing all work.
- Traffic is **encrypted on both sides**, so it is safe even inside the cluster.
- Good for strict security and compliance rules.

**Disadvantages**

- **Most CPU use**, because decrypt and encrypt both happen.
- **More certificates to manage** (load balancer and app).
- Slightly more delay and more complex setup.

Bridging gives you **both**: full load balancer features and end-to-end encryption

|                     | SSL Passthrough                                                  | SSL Offloading / Termination                              | SSL Bridge / Re-encrypt                                        |
| ------------------- | ---------------------------------------------------------------- | --------------------------------------------------------- | -------------------------------------------------------------- |
| Processing          | Costly in processing for server                                  | Fast in processing                                        | Costly in processing for server                                |
| Security            | Secure in many cases                                             | less secure                                               | Secure                                                         |
| Load balancing type | L4 (TCP) load balancing                                          | L7 load balancing                                         | L7 load balancing                                              |
| When to choose      | When you don't bother about access rules, blocking, cookies etc. | When you want less latency and can compromise on security | When you need security and advanced load balancer capabilities |
| Inspection          | No load balancer inspection                                      | Load balancer inspects the packets                        | Load balancer inspects the packets                             |
| Recommended?        | Recommended*                                                     | Highly unrecommended                                      | Recommended                                                    |

### 3.7.4 what is certificate?

A **certificate** is a digital ID card for a website. It proves "this server really is `myapp.com`", and it carries the **public key** used to start the encrypted connection.

- It is issued by a trusted authority (for example Let's Encrypt). That authority is called a **CA (Certificate Authority)**.
- It comes with a **private key**, which stays secret on the server. The certificate is public, but the private key must never be shared.
- It **expires**, so it must be renewed.

Real-life example: a certificate is like a **passport**. A trusted government (the CA) issues it, and others trust it because of that.
In Kubernetes, the certificate and private key are stored in a **Secret** (type `kubernetes.io/tls`), and the Ingress uses it.

### 3.7.5  What is TCP, L4, and L7?

TCP is the basic way two computers send data to each other reliably. It only moves the data. It does not understand what the data means. Think of it as a road.

Networks are described in layers (the OSI model). You only need two:

|              | **L4 (Layer 4)**                                       | L7 (Layer 7)                                   |
| ------------ | ------------------------------------------------------ | ---------------------------------------------- |
| Layer name   | Transport (TCP)                                        | Application (HTTP)                             |
| What it sees | IP address and port only                               | Full request: path, cookies, headers, hostname |
| Example      | Postman who reads only the **address** on the envelope | Postman who **opens the letter** and reads it  |
| Can do       | Simple forwarding                                      | Path routing, WAF, session routing             |
| Speed        | Faster, simpler                                        | Slower, smarter                                |

So in your table:

Passthrough = L4, because it cannot read the encrypted content. It only forwards TCP.
Offloading and Bridging = L7, because they decrypt the traffic, so they can read the HTTP request.


---

### What is OpenShift?

**OpenShift** is Red Hat's version of Kubernetes. It is Kubernetes plus extra features (a web console, stricter security, built-in tools). Companies often use it in production.

### What is a Route?

A **Route** is OpenShift's own way to expose an app to the outside, like Ingress in plain Kubernetes. In fact, Routes existed before Ingress. OpenShift's built-in router (based on HAProxy) is the "ingress controller" that reads Routes.