
### 1. **Prometheus**

Prometheus is an open-source systems monitoring and alerting toolkit

- An open-source **monitoring system and time-series database**.
- It **pulls (scrapes)** metrics over HTTP from a `/metrics` endpoint on each target, every few seconds.
- It stores each metric with a timestamp, so you get history.
- You query the data with **PromQL**.
- With **Alertmanager**, it can send alerts (email, Slack, etc.) when a rule is broken.

### 2. **Grafana**

- An open-source **visualization tool**.
- It doesn’t collect or store the metrics itself. It connects to a data source (Prometheus) and turns the data into **dashboards and graphs**.

**Simple analogy:** Prometheus is the sensor and logbook. Grafana is the screen that shows it nicely.

### 3. Why Kubernetes needs them

- Kubernetes is **dynamic**: pods are created, deleted, and moved all the time. Prometheus can use the Kubernetes API for **service discovery** and automatically find new pods to scrape.
- There are **many moving parts** (nodes, pods, deployments, control plane), and you need visibility into all of them.
- `kubectl top` only shows the **current** usage. Prometheus keeps **history**, so you can answer “what happened at 3 AM?”.
- we get **alerts** for things like pod crash loops, high memory, or a node going down.
- we get dashboards for **capacity planning** (do we need more nodes?).


```
Ravilinux:~$ minikube start --memory=3072

Ravilinux:~$ kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-7d764666f9-vqd97           1/1     Running   0             58s
kube-system   etcd-minikube                      1/1     Running   0             66s
kube-system   kube-apiserver-minikube            1/1     Running   0             64s
kube-system   kube-controller-manager-minikube   1/1     Running   0             66s
kube-system   kube-proxy-zs8np                   1/1     Running   0             58s
kube-system   kube-scheduler-minikube            1/1     Running   0             66s
kube-system   storage-provisioner                1/1     Running   1 (27s ago)   62s
```

Installs Helm. `--classic` gives it full system access, which Helm needs.
```
Ravilinux:~$ sudo snap install helm --classic
[sudo] password for ravindu: 
helm 4.3.1 from Snapcrafters✪ installed

Ravilinux:~$ helm version
version.BuildInfo{Version:"v4.3.1
```

**Install Prometheus in minikube**

```
Ravilinux:~$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories

Ravilinux:~$ helm repo update
Update Complete. ⎈Happy Helming!⎈
```

```
Ravilinux:~$ helm install prometheus prometheus-community/prometheus
NAME: prometheus
LAST DEPLOYED: Thu Oct 22 20:10:03 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.default.svc.cluster.local


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9090

Prometheus alertmanager can be accessed via port 9093 on the following DNS name from within your cluster:
prometheus-alertmanager.default.svc.cluster.local


Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093

Prometheus Pushgateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheus-prometheus-pushgateway.default.svc.cluster.local


Get the Pushgateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=prometheus-pushgateway,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/
```

```
Ravilinux:~$ kubectl get pods

NAME                                                 READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-0                            2/2     Running   0          16m
prometheus-kube-state-metrics-798fb7d7b8-f2jbx       1/1     Running   0          16m
prometheus-prometheus-node-exporter-fj652            1/1     Running   0          16m
prometheus-prometheus-pushgateway-5b797b4867-r7ztx   1/1     Running   0          16m
prometheus-server-67884b78bb-wzh46                   2/2     Running   0          16m
```

`helm` and `kubectl` are only **client tools** on your laptop. They don’t create pods themselves. They send requests to the **Kubernetes API server** of whichever cluster your kubeconfig points to.
When you ran `minikube start`, it automatically wrote the minikube cluster’s address and credentials into `~/.kube/config` and set minikube as the **current context**. Helm reads the same file.

What happened when you ran `helm install`


Laptop: helm reads chart → renders YAML files
        ↓ (uses ~/.kube/config to find the cluster)
Minikube API server: receives the YAML
        ↓
Scheduler + kubelet: create the pods inside minikube

```
ravindu@Ravilinux:~$ kubectl get svc
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes                            ClusterIP   10.96.0.1       <none>        443/TCP    35h
prometheus-alertmanager               ClusterIP   10.99.175.238   <none>        9093/TCP   33h
prometheus-alertmanager-headless      ClusterIP   None            <none>        9093/TCP   33h
prometheus-kube-state-metrics         ClusterIP   10.102.95.98    <none>        8080/TCP   33h
prometheus-prometheus-node-exporter   ClusterIP   10.105.67.128   <none>        9100/TCP   33h
prometheus-prometheus-pushgateway     ClusterIP   10.108.32.254   <none>        9091/TCP   33h
prometheus-server                     ClusterIP   10.99.162.171   <none>        80/TCP     33h
```

**Expose the service**
```
Ravilinux:~$ kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

Prometheus-server is type `ClusterIP`, so it is reachable only from inside the cluster. To open the UI in your browser, you need a `NodePort` service

```
Ravilinux:~$ kubectl get svc
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes                            ClusterIP   10.96.0.1       <none>        443/TCP        35h
prometheus-alertmanager               ClusterIP   10.99.175.238   <none>        9093/TCP       33h
prometheus-alertmanager-headless      ClusterIP   None            <none>        9093/TCP       33h
prometheus-kube-state-metrics         ClusterIP   10.102.95.98    <none>        8080/TCP       33h
prometheus-prometheus-node-exporter   ClusterIP   10.105.67.128   <none>        9100/TCP       33h
prometheus-prometheus-pushgateway     ClusterIP   10.108.32.254   <none>        9091/TCP       33h
prometheus-server                     ClusterIP   10.99.162.171   <none>        80/TCP         33h
prometheus-server-ext                 NodePort    10.100.68.104   <none>        80:30323/TCP   32s
```

`80:30323/TCP` means port 80 inside the cluster is opened on port **30323** on the node.

**Open the UI**
http://192.168.49.2:30323    (but On your setup (Ubuntu with Docker Desktop), that IP may not be reachable from your browser. Try this first:)

```
Ravilinux:~$ minikube service prometheus-server-ext --url
http://127.0.0.1:39267
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```


![[kubenetes-10-prometheus-login.png|514]]




**Install Grafana in mikube**

Registers the new Grafana repo on your laptop under the local name `grafana-community`:
```
Ravilinux:~$ helm repo add grafana-community https://grafana-community.github.io/helm-charts
"grafana-community" has been added to your repositories

Ravilinux:~$ helm repo update
Update Complete. ⎈Happy Helming!⎈

Ravilinux:~$ helm repo list
NAME                	URL                                               
prometheus-community	https://prometheus-community.github.io/helm-charts
grafana-community   	https://grafana-community.github.io/helm-

Ravilinux:~$ helm search repo grafana-community/grafana
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
grafana-community/grafana    	14.0.1       	13.2.4     	The leading tool for querying and visualizing t...
grafana-community/grafana-mcp	0.31.0       	2.1.1      	MCP server for Grafana.    

Ravilinux:~$ helm install grafana grafana-community/grafana
NAME: grafana
LAST DEPLOYED: Sat Oct 24 11:32:23 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
NOTES:
1. Get your 'admin' user password by running:

   kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
                     
```

**Getting password for login grafana**
```
Ravilinux:~$ kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
4peGY3RECNcUJXbgjhVtgs0uVlL05oHrbW2dcQ0n
```

**Expose grafana service  for outside using nodeport**
```
Ravilinux:~$ kubectl get svc
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
grafana                               ClusterIP   10.99.45.206    <none>        80/TCP         20m
kubernetes                            ClusterIP   10.96.0.1       <none>        443/TCP        41h
prometheus-alertmanager               ClusterIP   10.99.175.238   <none>        9093/TCP       39h
prometheus-alertmanager-headless      ClusterIP   None            <none>        9093/TCP       39h
prometheus-kube-state-metrics         ClusterIP   10.102.95.98    <none>        8080/TCP       39h
prometheus-prometheus-node-exporter   ClusterIP   10.105.67.128   <none>        9100/TCP       39h
prometheus-prometheus-pushgateway     ClusterIP   10.108.32.254   <none>        9091/TCP       39h
prometheus-server                     ClusterIP   10.99.162.171   <none>        80/TCP         39h
prometheus-server-ext                 NodePort    10.100.68.104   <none>        80:30323/TCP   6h15m

ravindu@Ravilinux:~$ kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
service/grafana-ext exposed

Ravilinux:~$ kubectl get svc
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
grafana                               ClusterIP   10.99.45.206    <none>        80/TCP         27m
grafana-ext                           NodePort    10.105.7.90     <none>        80:31027/TCP   2m38s
kubernetes                            ClusterIP   10.96.0.1       <none>        443/TCP        41h
prometheus-alertmanager               ClusterIP   10.99.175.238   <none>        9093/TCP       39h
prometheus-alertmanager-headless      ClusterIP   None            <none>        9093/TCP       39h
prometheus-kube-state-metrics         ClusterIP   10.102.95.98    <none>        8080/TCP       39h
prometheus-prometheus-node-exporter   ClusterIP   10.105.67.128   <none>        9100/TCP       39h
prometheus-prometheus-pushgateway     ClusterIP   10.108.32.254   <none>        9091/TCP       39h
prometheus-server                     ClusterIP   10.99.162.171   <none>        80/TCP         39h
prometheus-server-ext                 NodePort    10.100.68.104   <none>        80:30323/TCP   6h22m

Ravilinux:~$ kubectl get pods
NAME                                                 READY   STATUS    RESTARTS        AGE
grafana-6d48c47cbc-nt5wf                             1/1     Running   0               22m
prometheus-alertmanager-0                            2/2     Running   4 (7h35m ago)   40h
prometheus-kube-state-metrics-798fb7d7b8-f2jbx       1/1     Running   4 (7h34m ago)   40h
prometheus-prometheus-node-exporter-fj652            1/1     Running   2 (15h ago)     40h
prometheus-prometheus-pushgateway-5b797b4867-r7ztx   1/1     Running   2 (15h ago)     40h
prometheus-server-67884b78bb-wzh46                   2/2     Running   4 (7h35m ago)   40h
```
http://192.168.49.2: 31027 (but On your setup (Ubuntu with Docker Desktop), that IP may not be reachable from your browser. Try this first:)

**Create minikube tunnel**
```
Ravilinux:~$ minikube service grafana-ext --url
http://127.0.0.1:46661
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

![[kubernetes-10-grafana-login.png|578]]