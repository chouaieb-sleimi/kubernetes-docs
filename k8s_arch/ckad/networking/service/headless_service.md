# K8S Service - Headless

tags: #objects #network

---

https://kubernetes.io/docs/concepts/services-networking/service/#headless-services

in headless service: `.spec.clusterIP` = `"None"`.

- no service IP address is allocated,
- no load balancing
- kube-proxy does not handle these Services
- used for service discovery mechanisms (DNS names)

How DNS is automatically configured depends on whether the Service has selectors defined:

- w/out selectors
- w/ selectors
  - creates `EndpointSlices` in the api
  - configures DNS to return records directly to the Pods backing the Service.
    - records: A or AAAA (IPv4 or IPv6 addresses)

sample

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - port: 80
      name: web
```

## Headless Service w/ StatefulSet

## Headless Service w/ Deployment

> Note
> `subdomain` and `hostname` must be specified for DNS records to be assigned

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc-h
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
      # w/out hostname given, pods names:
      # mysql-h.default.svc.cluster.local
      subdomain: mysql-svc-h

      # w/ subdomain, pods names:
      # mysql-pod.mysql-h.default.svc.cluster.local
      hostname: mysql-pod
```

reslting pods DNS records:

- `mysql-pod.mysql-svc-h.my-ns.svc.local.cluster`
- `mysql-pod.mysql-svc-h.my-ns.svc.local.cluster`
- `mysql-pod.mysql-svc-h.my-ns.svc.local.cluster`
