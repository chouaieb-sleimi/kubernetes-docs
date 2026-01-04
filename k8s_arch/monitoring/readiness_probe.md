# K8S Readiness Probe

tags: #objects #monitoring

---

- are the [[containers]] is ready to respond to requests/accept trafic?
- **on probe fails:** the endpoints controller removes the Pod's IP address from the endpoints of all pod Services.
- default **state of readiness before the initial delay** is Failure. 
- **default readiness probe:** state is Success.

**example:** Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

probe types:

- **HTTP request test**
- **Port test** (TCP Socket)
- **Script execution**

**readiness probe** sample

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
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 10   # wait before probes begin
      periodSeconds: 5          # deplay bettween probes
      failureThreshold: 8       # retries before failure declared; default is 3

    # TCP Test
    readinessProbe:
      tcpSocket:
        port: 3306

    # Script execution
    readinessProbe:
      exec:
        command:
        - cat
        - /app/is_ready
```
