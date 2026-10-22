
. **Prometheus**

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
add helm repo
```
Ravilinux:~$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories

Ravilinux:~$ helm repo update
Update Complete. ⎈Happy Helming!⎈
```

Install Prometheus
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