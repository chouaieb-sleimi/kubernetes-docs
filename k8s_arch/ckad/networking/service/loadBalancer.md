# K8S Service - NodePort

tags: #objects #network

---

https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer

- provisions a load balancer for our service in supported cloud providers.
- cloud provider decides how trafic it is load balanced.


```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: MyApp
  clusterIP: 10.0.171.239
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```