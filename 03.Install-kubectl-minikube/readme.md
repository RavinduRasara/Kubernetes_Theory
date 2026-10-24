
### 1\. Install kubectl

**1.1 Download kubectl**

```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

**1.2 Check file is downloaded**

```bash
ls -lh kubectl
```

**1.3 Validate (optional but recommended)**

This is a **security check!**

```
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

```
echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
```

**1.4 Install kubectl**

```
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

```
$ kubectl version  
Client Version: v1.35.3  
Kustomize Version: v5.7.1  
Error from server (Forbidden): <html><head><meta http-equiv='refresh' content='1;url=/login?from=%2Fversion%3Ftimeout%3D32s'/><script id='redirect' data-redirect-url='/login?from=%2Fversion%3Ftimeout%3D32s' src='/static/6b9955ee/scripts/redirect.js'></script></head><body style='background-color:white; color:white;'>  
Authentication required  
<!--  
-->

</body></html>  

$ kubectl version --client  
Client Version: v1.35.3  
Kustomize Version: v5.7.1
```

### **2\. Install minikube**

```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

```

&nbsp;

```bash
$ minikube version
W0404 18:43:19.579618   55820 main.go:294] Unable to resolve the current Docker CLI context "default": context "default": context not found: open /home/ravindu/.docker/contexts/meta/37a8eec1ce19687d132fe29051dca629d164e2c4958ba141d5f4133a33f0688f/meta.json: no such file or directory
W0404 18:43:19.579655   55820 main.go:295] Try running `docker context use default` to resolve the above error
minikube version: v1.38.1
commit: c93a4cb9311efc66b90d33ea03f75f2c4120e9b0

```

**🤔 What is Docker Context?**

Simple explanation:

> **Docker context = tells Docker WHERE to connect to**

```
Docker can connect to:
→ Your local machine (default)
→ Remote server
→ Cloud server (AWS, GCP etc.)

Context = the address/location
          Docker is currently pointing to!
```

```
Docker was looking for context file:
/home/ravindu/.docker/contexts/meta/
37a8eec.../meta.json

This file = MISSING! ❌

Means Docker context got confused!
Does not know where to point!

"docker context use default"
= hey Docker! 
  forget confusion!
  just use LOCAL machine! ✅
```

&nbsp;

```
$ docker context use default
default
Current context is now "default"

$ minikube version
minikube version: v1.38.1
commit: c93a4cb9311efc66b90d33ea03f75f2c4120e9b0

```

### 3\. Start Minikube

```
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
💾  Downloading Kubernetes v1.35.1 preload ...
    > preloaded-images-k8s-v18-v1...:  272.45 MiB / 272.45 MiB  100.00% 126.10 
    > gcr.io/k8s-minikube/kicbase...:  519.58 MiB / 519.58 MiB  100.00% 135.15 
🔥  Creating docker container (CPUs=2, Memory=1870MB) ...
🐳  Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default


Ravilinux:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   26m   v1.35.1

```