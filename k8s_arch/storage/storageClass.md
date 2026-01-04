# K8S StorageClass

tags: #objects #storage

---

- dynamic provisioning of [[persistentVolume]];
- upon a claim, the storage is created.

StorageClass sample

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
volumeBindingMode: [ImmediateWaitForFirstConsumer]
parametes:
  types: [pd-standard | pd-ssd]
  replication-type: [none | regional-pd]
```

use StorageClass in PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: vol-claim-1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resourcecs:
    requests:
      storage: 500Mi
```

use PVC in pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-num-gen
  ...
  volumes:
  - name: data-volume
    persistentVolumeClaim:
        claimName: vol-claim-1
```
