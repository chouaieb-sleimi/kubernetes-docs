# K8S PersistentVolume

tags: #objects #storage

---

- **separate manifest required** to create a PV
- **separates storage from a Pod**
- cluster-wide storage
  - **statically provisioned** by admins
  - **dynamically provisioned** w/ `StorageClass`
- have a lifecycle independent of any Pods that uses PVs.
  - enables **safe Pod restart**
- binded to PVCs

**PV types:** [[volume_types]]


**PV `persistentVolumeReclaimPolicy`:**

- `Retain` (default)
  keep PV, keep contents
- `Delete`
  delete PV, delete contents
- `Recycle`
  keep PV, delete contents

> For dynamically provisioned PVs, default reclaim policy is `Delete`

**PV `accessModes`:**

- `ReadWriteOnce`
- `ReadOnlyMany`
- `ReadWriteMany`

PV sample

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Recycle

  hostPath: # node storage
    path: /tmp/data

  awsElasticBlockStore: # AWS storage (deprecated)
    volumeID: <colume-id>
    fsType: ext4
```
