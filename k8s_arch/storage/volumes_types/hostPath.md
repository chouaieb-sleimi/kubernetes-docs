# K8S hostPath Volume

tags: #storage

---

- mounts a file or directory from host node's filesystem
- supported **hostPath volume types:**
  - `DirectoryOrCreate`
  - `Directory`
  - `FileOrCreate`
  - `File`
  - `Socket`
  - `CharDevice`
  - `BlockDevice`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-num-gen
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out"]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```
