# K8S Role

tags: #security

---

creates role that can view, create, delete pods and create ConfigMaps

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer-role
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["list, "get", "create", "update", "watch", "delete"]
    resourceName: ["front-end", "back-end"]
  - apiGroups: [""]
    resources: ["ConfigMaps"]
    verbs: ["create"]
```
