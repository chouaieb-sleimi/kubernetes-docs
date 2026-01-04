# K8S local Volume

tags: #storage

---

- mounts a **local storage device**:
  - **directory**
  - **disk**
  - **partition**
- are only used as a statically created `PersistentVolume`,
  - **Dynamic provisioning not supported.**
- `nodeAffinity` must be set,
  - scheduler uses `nodeAffinity` field to schedule Pods to correct nodes.
- to expose volume as a raw block device: set `volumeMode` to `Block`

`PersistentVolume` using a `local` volume and `nodeAffinity` sample

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - example-node
```
