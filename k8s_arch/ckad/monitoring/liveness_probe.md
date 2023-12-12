# K8S Liveness Probe

tags: #objects #monitoring

---

are apps in [[containers]] running/healthy?.

- **liveness probe fails:**
  - kubelet **kills the container**,
  - containers **restart policy** is applied.
- **default liveness probe:** state is Success.

**example:** catch deadlocks, where an application is running, but unable to make progress.

**Liveness probe types:**

- HTTP request test
- Port test
- Script execution

**liveness probe** sample

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: simple-webapp

spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080

    # HTTP Test
    livenessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 8     # default is 3

    # TCP Test
    livenessProbe:
      tcpSocket:
        port: 3306

    # Script execution
    livenessProbe:
      exec:
        command:
        - cat
        - /app/is_ready
```
