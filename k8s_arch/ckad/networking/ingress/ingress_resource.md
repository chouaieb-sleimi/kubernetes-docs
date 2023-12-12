# Ingress Resources

tags: #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Ingress Resources](#ingress-resources)
  - [Single URL - Single paths - Single backend](#single-url---single-paths---single-backend)
  - [Single URL - Mutliple paths - Mutliple backends](#single-url---mutliple-paths---mutliple-backends)
  - [Multiple URLs - Mutliple backends](#multiple-urls---mutliple-backends)

<!-- /code_chunk_output -->

---

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

```

```bash
# format
kubectl create ingress <ingress-name> --rule="host/path=service:port"

# example
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

## Single URL - Single paths - Single backend

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

## Single URL - Mutliple paths - Mutliple backends

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

## Multiple URLs - Mutliple backends

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
