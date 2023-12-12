# K8S Secret

tags: #objects #configuration

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Secret](#k8s-secret)
  - [Consume Secret](#consume-secret)

<!-- /code_chunk_output -->

---

- not encrypted, only encoded
- secrets are not encrypted in ETCD
  - configure encryption at rest (they are stored encrypted in ETCD)
    see: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
- anyone that cat create pods/deployments can see secrets
  - configure least-privilege access to secrets - RBAC
- consider third-party secrets store providers (AWS, Azure, GCP, Vault, etc.)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-secrets
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Pwd: cGFzc3dvcmQ=
```

## Consume Secret

consume **single variable**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          env:
            - name: APP_COLOR
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: DB_Host

consume **secret**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          envFrom:
            - secretRef:
                name: app-secrets

consume **secret from volume**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          volumes:
            - name: app-secrets-volume
              secret:
                secretName: app-secrets
