# K8S Ingress

tags: #objects #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Ingress Controller](#ingress-controller)
- [Ingress Resources](#ingress-resources)
  - [Redirect HTTP to HTTPS](#redirect-http-to-https)
  - [Single URL - Single paths - Single backend](#single-url---single-paths---single-backend)
  - [Single URL - Mutliple paths - Mutliple backends](#single-url---mutliple-paths---mutliple-backends)
  - [Multiple URLs - Mutliple backends](#multiple-urls---mutliple-backends)
  - [Rewrite Target](#rewrite-target)

<!-- /code_chunk_output -->

---

- exposes HTTP and HTTPS **routes from outside the cluster to services within the cluster.**
- traffic routing by rules in Ingress resource.
- provides:
  - load balancing
  - SSL termination
  - name-based virtual hosting
- **ingress limitations**
  - no support for:
    - multi-tenancy
    - namespace isolation
    - no RBAC for features
    - no resource isolation
    - TCP/UDP routing
    - traffic splitting/weighting
    - header manipulation
    - authentication
    - rate limiting
    - custom error pages
    - session affinity

**Ingress components:**

- **Ingress controller** (Ingress runtimes)
  a special LoadBalancer container deployment customized for Ingress. They can be:
  - Cloud LoadBalancers (supported)
  - Nginx (supported)
  - HAProxy
  - traefik
  - Istio
  - Contour

- **Ingress resources**
  Routing rules for the ingress controller

## Ingress Controller

[[ingress-controller]]

## Ingress Resources

see:
[kubernetes.io - create ingress command ref](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-)
[kubernetes.io - ingress](https://kubernetes.io/docs/concepts/services-networking/ingress)
[kubernetes.io - ingress examples](https://kubernetes.github.io/ingress-nginx/examples/)

> Note: ingress resource apiVersion may change depending on k8s' version

**Basic routing components**

- Single/Mutliple URLs
- Single/Mutliple paths
- Single/Mutliple backend services

get ingress resources

```bash
kubectl get ingress --namespace <namespace-name>
```

create ingress imperatively

```bash
# format
kubectl create ingress <ingress-name> --rule="host/path=service:port"

# example
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

### Redirect HTTP to HTTPS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    # Explicitly enforce the HTTP to HTTPS redirect
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    # Force redirect even if no TLS block is present (e.g., if SSL terminates at an external load balancer)
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - example.com
      secretName: example-tls-secret
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

### Single URL - Single paths - Single backend

schema

```
# URL: www.my-online-store.com
    Path: /
    backend sevice: wear-service
```

ingress object sample

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear

spec:
  backend:
    name: wear-service
    port:
      number: 80
```

### Single URL - Mutliple paths - Mutliple backends

schema

```
# URL: www.my-online-store.com

    Path: /wear
    backend service: wear-service

    Path: /watch
    backend service: watch-service

    # default 404 page
    Path: *
    backend service: default-http-backend
```

ingress object sample

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch

spec:
  rules:
    - http:
        paths:
          - path: /wear
            backend:
              service:
                name: wear-service
                port:
                  number: 80

          - path: /watch
            backend:
              service:
                name: watch-service
                port:
                  number: 80

          # default 404 page
          - path: /
            backend:
              service:
                name: default-http-backend
                port:
                  number: 80
```

### Multiple URLs - Mutliple backends

schema

```
# URL: www.wear.my-online-store.com

    Path: *
    backend service: wear-service

# URL: www.watch.my-online-store.com

    Path: *
    backend service: watch-service
```

ingress object sample

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch

spec:
  rules:
    - host: www.wear.my-online-store.com
      http:
        paths:
          - backend:
              service:
                name: wear-service
                port:
                  number: 80

    - host: www.watch.my-online-store.com
      http:
        paths:
          - backend:
              service:
                name: watch-service
                port:
                  number: 80

          # default 404 page
          - path: /
            backend:
              service:
                name: default-http-backend
                port:
                  number: 80
```

### Rewrite Target

rewrites `ingress-url/pay` > `service/`

- instead of `ingress-url/pay` > `service/pay`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-pay
  annotations:
    nginx.ingress.kubernetes.io/rerite-target: /

spec:
  rules:
    - host: my-pay.website
      http:
        paths:
          - path: /pay
            backend:
              service:
                name: wear-service
                port:
                  number: 80
```

- rewrites `rewrite.bar.com/something` > `rewrite.bar.com/`
- rewrites `rewrite.bar.com/something/` > `rewrite.bar.com/`
- rewrites `rewrite.bar.com/something/new` > `rewrite.bar.com/new`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-pay
  annotations:
    nginx.ingress.kubernetes.io/rerite-target: /$2

spec:
  rules:
    - host: rewrite.bar.com
      http:
        paths:
          - path: /something(/|$)(.*)
            backend:
              service:
                name: wear-service
                port:
                  number: 80
```
