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


