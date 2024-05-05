# K8S volumeClaimTemplates

tags: #storage

---

- a template for [[persistentVolumeClaim]] defined in a [[statefulSet]]
- generated PVCs claims PVs provisioned by a storageClass
- `volumeClaimTemplates` provide stable storage using PVs provisioned by a PV Provisioner.

On **pod failure/reschedules, volumeClaimTemplates-v PVCs are not removed** but are **instead attached to the recreated pods**

sample 1

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  ...
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

sample 2

```yaml
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
