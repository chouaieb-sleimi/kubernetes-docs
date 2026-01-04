# K8S configMap Volume

tags: #storage

---

- **purpose:** inject configuration data into pods

sample

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      command: ["sh", "-c", 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```

- `log-config` ConfigMap is mounted as a volume
- all contents stored in `log_level` entry are mounted at path `/etc/config/log_level` (`mountPath: /etc/config `+ `path: log_level`)

> Note:
>
> - ConfigMap must exist before use.
> - ConfigMap is always mounted as `readOnly`.
> - Text data files use UTF-8 character encoding. use `binaryData` for others.
