# K8S emptyDir Volume

tags: #storage

---

- volume is created when Pod is assigned to a node.
- volume is initially empty.
- containers in Pod can read/write the same files in the volume
  - volume can be mounted at the **same or different paths** in each container.
- when a **Pod is removed from a node** for any reason,
  - volume **data is deleted permanently**.

> Note: A container crashing does not remove a Pod from a node. The data in an emptyDir volume is safe across container crashes.

sample

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
    - image: registry.k8s.io/test-webserver
      name: test-container
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir:
        sizeLimit: 500Mi
```
