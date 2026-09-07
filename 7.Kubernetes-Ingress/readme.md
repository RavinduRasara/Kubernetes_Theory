
#### Kubenetes Ingress

**![ingress1.png](/7.Kubernetes-Ingress/img/ingress1.png)**


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

```
Kubernetes-Ingress$ kubectl apply -f ingress.yml
ingress.networking.k8s.io/ingress-example created

Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS   PORTS   AGE
ingress-example   <none>   foo.bar.com             80      28s

Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS   PORTS   AGE
ingress-example   <none>   foo.bar.com             80      28s

```

install nginx ingress controller on minikube cluster

```
Kubernetes-Ingress$ minikube addons enable ingress
```

End of the ingress controller also pods.to check 

```
Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS   PORTS   AGE
ingress-example   <none>   foo.bar.com             80      28s

```

```
Kubernetes-Ingress$ kubectl get pods -A | grep nginx
ingress-nginx   ingress-nginx-admission-create-vr8pn        0/1     Completed   0               87m
ingress-nginx   ingress-nginx-admission-patch-4dz65         0/1     Completed   0               87m
ingress-nginx   ingress-nginx-controller-596f8778bc-nd8jj   1/1     Running     0               87m
```

identify ingress resource we create

```
Kubernetes-Ingress$ kubectl logs ingress-nginx-controller-596f8778bc-nd8jj -n ingress-nginx
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.14.3
  Build:         b4ab41015421ae27f3a96d73f013183b7e166735
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.27.1

-------------------------------------------------------------------------------
```

```
Kubernetes-Ingress$ kubectl get ingress
NAME              CLASS    HOSTS         ADDRESS        PORTS   AGE
ingress-example   <none>   foo.bar.com   192.168.49.2   80      4h8m
```

 we have to update etc/host configuration. 
 
 because  we are doing in local and we have not done domain mapping. (foo.bar.com -192.168.49.2) .192.168.49.2 is ingress ip address. foo.bar.com not real domain.
 we can buy domain from GoDaddy  in real production and can it use for this.
 
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