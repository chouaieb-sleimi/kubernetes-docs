# K8S ConfigMap

tags: #objects #configuration

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Environment Variables](#environment-variables)
- [Consume Config Map](#consume-config-map)

<!-- /code_chunk_output -->

---

consumeed as:

- environment variables,
- command-line arguments,
- as configuration files in a volume.

## Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata: ...
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: red
```

Value Types:

- **Plain Key Value pair:**

```
env:
  - name: APP_COLOR
    value: red
```

- **ConfigMap:**

```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        ...
```

- **Secrets:**

```
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        ...
```

## Consume Config Map

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
                configMapKeyRef:
                  name: app-config-map
                  key: APP_COLOR

consume **configmap**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          envFrom:
            - configMapRef:
                name: app-config-map

consume **configmap from volume**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          volumes:
            - name: app-config-volume
              configMap:
                name: app-config-map
