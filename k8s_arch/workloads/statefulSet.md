# K8S StatefulSets

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->

---

for applications that require one or more of the following:

- stable, unique network identifiers.
- stable, persistent storage.
- ordered, graceful deployment and scaling.
- ordered, automated rolling updates.
- support [[rollout_rollback]]

allows you to relax ordering guarantees with `.spec.podManagementPolicy` field it can be:

- `OrderedReady` (default).
- `Parallel`

> StatefulSets **must** be configured with a headless service for DNS naming

headless service and stateful set sample

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-h-svc
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: nginx

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  replicas: 3 # by default is 1
  serviceName: nginx-h-svc
  #podManagementPolicy: Parallel | OrderedReady
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      containers:
        - name: nginx
          image: registry.k8s.io/nginx-slim:0.8
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "my-storage-class"
        resources:
          requests:
            storage: 1Gi
```

reslting pods DNS records:

- `nginx-0.nginx-h-svc.my-ns.svc.local.cluster`
- `nginx-1.nginx-h-svc.my-ns.svc.local.cluster`
- `nginx-2.nginx-h-svc.my-ns.svc.local.cluster`
