# K8S Security Context

tags: #objects #security

---

applied at the level of [[containers]] and [[pod]]

- **pod level** security context.

  - _applies to all containers_
  - _capabilities **only available in container level**_

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  securityContext:
    runAsUser: 1012
  containers:
    - name:
      ...
```

- **container level** security context. _if pod security context exist, the container context is applied_

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name:
      ...
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
          drop:
            - KILL
```
