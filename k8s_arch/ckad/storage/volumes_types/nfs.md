# K8S nfs Volume

tags: #storage

---

- mounts an existing NFS share (Network File System)
- when a **pod is removed,**
  - volume **contents are preserved,**
    - **volume unmounted.**
- can be mounted by a **multiple read-write** consumers.
- NFS **mount options not allowed** in a Pod spec.
  - either set mount options server-side
  - or use `/etc/nfsmount.conf`.
- can be mounted via `PersistentVolumes`
  - **mount options allowed**

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
        - mountPath: /my-nfs-data
          name: test-volume
  volumes:
    - name: test-volume
      nfs:
        server: my-nfs-server.example.com
        path: /my-nfs-volume
        readOnly: true
```
