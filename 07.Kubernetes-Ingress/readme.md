
#### Kubenetes Ingress


![[ingress1.png]]

### 1.What is Ingress? 



>  Ingress is a Kubernetes API object that manages external access to services inside a cluster, typically for HTTP and HTTPS traffic.


- ClusterIP is for internal communication only. NodePort exposes a service externally but becomes unmanageable and expensive at scale because each service requires its own port or Load Balancer. Ingress solves this by acting as a smart Layer 7 (HTTP/HTTPS) router. It provides a **single external entry point** and routes traffic to multiple internal ClusterIP services based on hostnames or URL paths, making it much more efficient and cost-effective

- Everything you create in Kubernetes (Pods, Deployments, Services, and now Ingress) is an API object. When you write a YAML file and run `kubectl apply -f`, you are sending a request to the Kubernetes API to create that object and save it in the cluster's database.

---
### 2. Why does Ingress have to be created?

**2.1 The "Advanced Routing" Issue (Layer 4 vs. Layer 7)**

A standard Kubernetes Service (ClusterIP, NodePort, or LoadBalancer) is "dumb" when it comes to web traffic. It operates at **Layer 4 (TCP/UDP)**. It only knows IP addresses and Ports. It _cannot_ read the actual web request.

Enterprise load balancers (like Nginx or AWS ALB) operate at **Layer 7 (HTTP/HTTPS)**. They can read the request and make smart decisions. Before Ingress, Kubernetes couldn't do this natively.

- Path-based routing: Send `/api` to Service A, and `/web` to Service B. (A basic Service can't do this).
- Domain-based routing: Send `shop.amazon.com` to Service A, and `aws.amazon.com` to Service B.
- Sticky sessions: Ensure a specific user always goes to the same Pod (useful for shopping carts).
- Ratio-based (Canary): Send 90% of traffic to the old version, and 10% to the new version to test it.

Ingress was created to bring these "smart" Layer 7 routing capabilities into Kubernetes.

**2.2 The "Cost and IP" Issue (The Amazon 100 Microservices Example)**

Imagine Amazon has 100 different micro-services  (Cart, Search, Login, Payments, etc.), and they want all of them accessible from the internet.

- **The Old Way (LoadBalancer Service):** If they use `type: LoadBalancer` for each service, Kubernetes asks AWS to create **100 separate Cloud Load Balancers**.

    - Each Load Balancer gets its own Static Public IP.
    - Cloud providers charge you **per Load Balancer** (e.g., ~$25/month each).
    - **Result:** 100 Load Balancers = $2,500/month, plus wasting 100 Public IPs. This is incredibly expensive and messy to manage.

- **The Ingress Way:** Amazon creates just **ONE** Cloud Load Balancer (1 Public IP, ~$25/month). Behind it, they deploy **one Ingress Controller**. The Ingress Controller reads the incoming web traffic and smartly routes it to the correct internal ClusterIP Service (Cart, Search, Login, etc.).

    - **Result:** 1 Load Balancer = $25/month. Massive cost savings and only 1 IP to manage.


---
605### 3. Does kubernetes have a default load-balancer before Ingress?

Short Answer: No, Kubernetes itself does not have a built-in load balancer.

Simple Explanation: Kubernetes is "cloud-agnostic" (it doesn't care if it runs on AWS, Google Cloud, or your local laptop).

- When you create a `type: LoadBalancer` Service, Kubernetes simply sends a request to your **Cloud Provider** (like AWS or GCP) saying, _"Please create a load balancer for me and give me the IP."_ The cloud provider then creates it.

---
### 3. **Three ways** people did it before Ingress:

1. **The "One Load Balancer Per Service" Way (The Expensive Way)**

   This is what we talked about earlier with the Amazon example.

- How it worked: For every single microservice, developers created a Service with `type: LoadBalancer`.
- The Problem: If you had 100 microservices, the cloud provider created 100 separate Load Balancers. It worked perfectly, but it cost a fortune and wasted hundreds of Public IPs.

2. **The "NodePort + External Load Balancer" Way (The Clunky Way)**

- How it worked: Developers created Services with `type: NodePort`. This opened a random high port (like `30080`) on every Kubernetes node. Then, they had to manually set up a traditional, external Load Balancer (like an AWS Classic Load Balancer or a physical F5 hardware load balancer) and point it to the IP addresses of the Kubernetes nodes and those specific ports.
- The Problem: If a Kubernetes node crashed and a new one was created with a different IP, the external Load Balancer configuration had to be manually updated. It was a maintenance nightmare.

3. **The "Manual External Nginx" Way (The Painful Way)**

- **How it worked:** Developers would spin up a separate virtual machine outside the cluster and install Nginx or HAProxy on it. They would manually write Nginx configuration files to route `domain.com/api` to a NodePort, and `domain.com/web` to another NodePort.
- **The Problem:** Every time a service changed, scaled, or moved, someone had to manually log into that Nginx server, edit the text file, and restart Nginx. It completely defeated the purpose of Kubernetes automation.


---
### 4. Practical 1  (path base)

ingress.yml (ingress resource)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: python-django-app-service
            port:
              number: 80
```

- What it does:* Defines the routing rule. It tells Kubernetes: "If a request comes for `foo.bar.com` and the path is `/bar`, send it to the `python-django-app-service` on port 80.".
- The Ingress Controller reads this file, sees the name `python-django-app-service`, and says: "Okay, I will send traffic to whatever Service has this exact name
- The Ingress YAML  says: "Send traffic to this service on its internal port 80.

```
Kubernetes-Ingress$ kubectl apply -f ingress.yml
ingress.networking.k8s.io/ingress-example created

Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS   PORTS   AGE
ingress-example   <none>   foo.bar.com             80      28s

```
`ADDRESS` column is **empty** and `CLASS` is `<none>`. This is because the rule exists in the database, but there is no Ingress Controller installed yet to actually read it and assign an IP._

It tells Minikube to download and install the **NGINX Ingress Controller**.
```
Kubernetes-Ingress$ minikube addons enable ingress
```

Checks the status. It might still show an empty address for a few seconds while the controller is starting up and reading the rules.
```
Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS   PORTS   AGE
ingress-example   <none>   foo.bar.com             80      28s

```

Verifies that the Ingress Controller is actually running
```
Kubernetes-Ingress$ kubectl get pods -A | grep nginx
ingress-nginx   ingress-nginx-admission-create-vr8pn        0/1     Completed   0               87m
ingress-nginx   ingress-nginx-admission-patch-4dz65         0/1     Completed   0               87m
ingress-nginx   ingress-nginx-controller-596f8778bc-nd8jj   1/1     Running     0               87m
```

Checks the logs of the controller pod.
```
Kubernetes-Ingress$ kubectl logs ingress-nginx-controller-596f8778bc-nd8jj -n ingress-nginx
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.14.3
  Build:         b4ab41015421ae27f3a96d73f013183b7e166735Ingress is a Kubernetes API object that manages external access to services inside a cluster, typically for HTTP and HTTPS traffic.
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.27.1

-------------------------------------------------------------------------------
```

Checks the Ingress status again.
```
Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS        PORTS   AGE
ingress-example   <none>   foo.bar.com   192.168.49.2   80      4h8m
```
**`192.168.49.2`**. This is your Minikube node's IP address. It proves the NGINX Controller has successfully read your rule, assigned an IP, and is actively listening for traffic.

```
Kubernetes-Ingress$ kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP        48d
python-django-app-service   NodePort    10.111.147.227   <none>        80:30007/TCP   48d
```
- `80:30007` is coming from your **previous Services tutorial**, not from the `ingress.yml` file.
- The **Ingress Controller** ignores the NodePort (`30007`) completely. In fact, it proves a very important concept
- It talks directly to the Service's internal `CLUSTER-IP` (`10.111.147.227`) on the internal port (`80`).
- The `30007` port is just sitting there. If someone wanted to bypass the Ingress, they _could_ use `192.168.49.2:30007`


** Need special update**
Django receives a request for `/bar`. But Django application doesn't have a `/bar` route, only `/demo/`. so need this update

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /demo/
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: python-django-app-service
            port:
              number: 80
```

**`rewrite-target` is an NGINX Ingress feature that lets you say: _"match incoming requests on this path, but before sending it to the pod, rewrite the path to something else. So we can match /bar externally, but internally rewrite it to /demo/"_**

```
 kubectl apply -f ingress.yml
```

Confirm the update took effect:
```
Kubernetes-Ingress$ kubectl describe ingress ingress-example
Name:             ingress-example
Labels:           <none>
Namespace:        default
Address:          192.168.49.2
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com  
               /bar   python-django-app-service:80 (10.244.0.50:8000,10.244.0.48:8000)
Annotations:   nginx.ingress.kubernetes.io/rewrite-target: /demo/

```

curl -L http://foo.bar.com/bar -v

in the production environment this is enough. curl -L http://foo.bar.com/bar -v

`-L` → **follow redirects**
 `v` → **verbose**
 
 

---
In local  minikube step: 

we have to update etc/host configuration. 
 
 because  we are doing in local and we have not done domain mapping. (foo.bar.com -192.168.49.2) .192.168.49.2 is ingress ip address. foo.bar.com not real domain.
 we can buy domain from GoDaddy  in real production and can it use for this). (your own machine fake DNS resolution for local testing domains)
 
```
 Kubernetes-Ingress$ sudo vim /etc/hosts
```

```
# End of section
192.168.49.2 foo.bar.com
```
we say , tell the machine  this domain (foo.bar.com) will be resolved on this specific ip address. (192.168.49.2). this something like mimic the behavior.  this is not production use case. in production we can  use directly domain name.

```
Kubernetes-Ingress$ ping foo.bar.com
PING foo.bar.com (192.168.49.2) 56(84) bytes of data.
```

```
Kubernetes-Ingress$ curl -L http://foo.bar.com/bar -v
* Host foo.bar.com:80 was resolved.
* IPv6: (none)
* IPv4: 192.168.49.2
*   Trying 192.168.49.2:80...

```

Terminal 1:
```
7.Kubernetes-Ingress$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress ingress-example requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🔗  Starting tunnel for service ingress-example.
[sudo] password for ravindu: 

```

Terminal 2: 
```
7.Kubernetes-Ingress$ curl -L http://foo.bar.com/bar -v
* Host foo.bar.com:80 was resolved.
* IPv6: (none)
* IPv4: 127.0.0.1
*   Trying 127.0.0.1:80...
* Connected to foo.bar.com (127.0.0.1) port 80
> GET /bar HTTP/1.1
> Host: foo.bar.com
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>

```
your laptop → tunnel → minikube node → NGINX Ingress Controller → Service → Pod → Django


### 5. Practical 2

