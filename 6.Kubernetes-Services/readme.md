
**1.Start minikube**

```bash
ravindu@Ravilinux:~$ minikube start
😄  minikube v1.38.1 on Ubuntu 24.04

ravindu@Ravilinux:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

ravindu@Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   2m17s   v1.35.1

ravindu@Ravilinux:~$ kubectl get pods
No resources found in default namespace.

ravindu@Ravilinux:~$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m52s

ravindu@Ravilinux:~$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   3m13s
kube-node-lease   Active   3m13s
kube-public       Active   3m13s
kube-system       Active   3m13s

```

**Project Folder**

```
Kubernetes-Services$ ls
deployment.yml  devops  Dockerfile  readme.md  requirements.txt  service.yml
```

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
python-sample-app   2/2     2            2           175m

6.Kubernetes-Services$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
python-sample-app-5f95f8b87d-nglqg   1/1     Running   0          175m   10.244.0.5   minikube   <none>           <none>
python-sample-app-5f95f8b87d-srjx7   1/1     Running   0          175m   10.244.0.4   minikube   <none>           <none>

6.Kubernetes-Services$ kubectl get rs
NAME                           DESIRED   CURRENT   READY   AGE
python-sample-app-5f95f8b87d   2         2         2       176m


```

&nbsp;

### **Kubernetes Service**

* * *

**<span style="color: rgb(255, 255, 255);">1.Create service</span>**

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
6.Kubernetes-Services$ kubectl apply -f service.yml
service/python-django-app-service created

```

```
6.Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes                  ClusterIP   10.96.0.1     <none>        443/TCP        3h23m
python-django-app-service   NodePort    10.97.46.33   <none>        80:30007/TCP   3s


```

```
6.Kubernetes-Services$ minikube ip
192.168.49.2
```

**![t1.png](/6.Kubernetes-Services/img/t1.png)**

* * *

**1\. ClusterIP** <span style="color: rgb(45, 194, 107);">—</span> *internal only*

<span style="color: rgb(45, 194, 107);">This is a **virtual IP inside the cluster** that nobody outside can reach.</span>

```
6.Kubernetes-Services$ kubectl get svc
NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
python-django-app-service   NodePort    10.97.46.33   <none>        80:30007/TCP   3s
```

<span style="color: rgb(45, 194, 107);">That `10.97.46.33` is the ClusterIP. Other apps *inside* the cluster can talk to your Django app using this IP on port 80. You on your laptop cannot reach it directly.</span>

<span style="color: rgb(53, 152, 219);">The column is called `CLUSTER-IP` but it does **not** mean "this service is of type ClusterIP". It means something different — it means **"this is the internal IP address assigned to this service inside the cluster"**.</span>

<span style="color: rgb(45, 194, 107);">**2\. NodePort** — *this is what you are using*</span>

<span style="color: rgb(45, 194, 107);">NodePort **opens a specific port on the node itself** so the outside world can reach your app. Your minikube node IP is `192.168.49.2`, and you opened port `30007`. So you can visit:</span>

```
http://192.168.49.2:30007
```

<span style="color: rgb(45, 194, 107);">The port journey is: `30007` (you knock on the node) → `80` (service receives it) → `8000` (Django app inside the container).</span>

```
port: 80        # ClusterIP port (inside cluster)
targetPort: 8000 # port your Django app listens on
nodePort: 30007  # port opened on the node for outside access
```

### Summary table

| Type | Who can reach it | Your example |
| --- | --- | --- |
| ClusterIP | Only other apps inside the cluster | `10.97.46.33:80` |
| NodePort | Anyone who can reach the node IP | `192.168.49.2:30007` |
| LoadBalancer | The whole internet | Not available in minikube |

<span style="color: rgb(45, 194, 107);">`nodePort: 30007` — this is the door on the minikube node itself. When you type `192.168.49.2:30007` in your browser, you are knocking on this door. NodePort values must always be between **30000 and 32767** — Kubernetes reserves this range specifically for NodePort services.</span>

<span style="color: rgb(45, 194, 107);">`port: 80` — this is the port the Service listens on *inside* the cluster. Once the request enters through the NodePort door, it is handed off to the Service at port 80. This is why other apps inside the cluster could also reach your Django app using `10.97.46.33:80`.</span>

<span style="color: rgb(45, 194, 107);">The full journey every time you refresh your browser is: `your laptop → 192.168.49.2:30007 → service :80 → pod :8000 → Django responds`.</span>

* * *

&nbsp;

<span style="color: rgb(224, 62, 45);">1\. Access application from inside cluster **(using ClusterIP service)**</span>

<span style="color: rgb(255, 255, 255);">using cluster ip *not cluster service*" but `10.97.46.33` IS the ClusterIP service address</span>

```
6.Kubernetes-Services$ minikube ssh
docker@minikube:~$ curl http://10.97.46.33:80/demo -L

<section>
  <nav>
    <ul>
      <li><a href="www.youtube.com/@AbhishekVeeramalla">YouTube</a></li>
      <li><a href="www.linkedin.com/in/abhishek-veeramalla-77b33996/">LinkedIn</a></li>
      <li><a href="https://telegram.me/abhishekveeramalla">Telegram</a></li>
    </ul>
  </nav>
</section>
<footer>
  <p>@AbhishekVeeramalla</p>
</footer>

</body>
</html>

```

`curl http://10.97.46.33:80/demo` worked because you were inside the cluster hitting the **ClusterIP** directly. This is exactly what ClusterIP means — only reachable from inside the cluster.

&nbsp;

<span style="color: rgb(224, 62, 45);">2\. Access application from inside cluster **(using Pod IP directly, bypassing the service)**</span>

```bash
6.Kubernetes-Services$ minikube ssh
docker@minikube:~$ curl -L http://10.244.0.4:8000/demo 
```

```
<section>
  <nav>
    <ul>
      <li><a href="www.youtube.com/@AbhishekVeeramalla">YouTube</a></li>
      <li><a href="www.linkedin.com/in/abhishek-veeramalla-77b33996/">LinkedIn</a></li>
      <li><a href="https://telegram.me/abhishekveeramalla">Telegram</a></li>
    </ul>
  </nav>
</section>



```

**`curl http://10.244.0.4:8000/demo` worked because you were hitting the **pod IP** directly, bypassing the service completely. This shows your Django app is healthy and running perfectly.**

&nbsp;

<span style="color: rgb(224, 62, 45);">3.Access application from outside cluster (*Use NodePort service)*</span>

```
6.Kubernetes-Services$ curl -L htttp://192.168.49.2:30007/demo
```

<span style="color: rgb(45, 194, 107);">In virtual machine or mackos we can access application directly using this command. **curl -L htttp://192.168.49.2:30007/demo (Without entering minikube)**</span>

<span style="color: rgb(45, 194, 107);">**Outside cluster → hits **NodePort** (30007) on the node → forwards to **ClusterIP service** (10.97.46.33:80) → ClusterIP service load balances to a **Pod** (:8000)**</span>

<span style="color: rgb(45, 194, 107);">**NodePort doesn't replace the ClusterIP service. It just adds an extra entry door on the node for outside traffic. Once the request enters through that door, it still goes through the ClusterIP service exactly like internal traffic does.**</span>

<span style="color: rgb(45, 194, 107);">**But in linux**</span>

**Teriminal 1**

```
6.Kubernetes-Services$ minikube service python-django-app-service --url
http://127.0.0.1:33241
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.

```

**Terminal 2**

```
6.Kubernetes-Services$ curl -L http://127.0.0.1:33241/demo

<section>
  <nav>
    <ul>
      <li><a href="www.youtube.com/@AbhishekVeeramalla">YouTube</a></li>
      <li><a href="www.linkedin.com/in/abhishek-veeramalla-77b33996/">LinkedIn</a></li>
      <li><a href="https://telegram.me/abhishekveeramalla">Telegram</a></li>
    </ul>
  </nav>
</section>

<footer>
  <p>@AbhishekVeeramalla</p>
</footer>

```

Its also work in browser - <span style="color: rgb(45, 194, 107);">http://127.0.0.1:33241/demo/</span>

Access application from inside cluster **(using ClusterIP service)**

**![t2.png](/6.Kubernetes-Services/img/t2.png)**

**127.0.0.1:33241 → tunnel → 192.168.49.2:30007 → port 80 → 10.97.46.33 (ClusterIP service) → pod**

**summary**:

1.  Inside cluster → using **ClusterIP service** → `10.97.46.33:80`
2.  Inside cluster → using **Pod IP directly** → `10.244.0.4:8000`
3.  Outside cluster → using **NodePort service** → `192.168.49.2:30007` (Mac/VM) or `127.0.0.1:33241` (Linux Ubuntu with tunnel)