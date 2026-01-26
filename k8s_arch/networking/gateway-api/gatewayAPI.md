# K8S Gateay API

tags: #objects #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Gateay API](#k8s-gateay-api)
    - [GatewayClass](#gatewayclass)
    - [Gateway](#gateway)
    - [Routes](#routes)
      - [HTTPRoute](#httproute)

<!-- /code_chunk_output -->

---

componenets:

- **[[GatewayClass]]**
  **owner:** infra provider
  defines underlying technology (cloud, nginx, traefik, etc.)

- **[[Gateway]]**
  **owner:** cluster operator
  instance of gateway class

- **[[routes]]**
  **owner:** application developers
  define how requests are routed to services
  - HTTPRoute
  - TCPRoute
  - UDPRoute
  - TLSRoute
  - GRCRoute

### GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: nginx-gateway-class
spec:
  controllerName: nginx.org/nginx-gateway-controller
  description: "NGINX Gateway Controller"
```

### Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: web-gateway
  namespace: default
spec:
  gatewayClassName: nginx-gateway-class
  listeners:
    - name: http
      port: 80
      protocol: HTTP
    - name: https
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - name: web-tls-cert
```

### Routes

#### HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: default
spec:
  parentRefs:
    - name: web-gateway
  hostnames:
    - "example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: "/api"
      backendRefs:
        - name: api-service
          port: 8080
    - matches:
        - path:
            type: PathPrefix
            value: "/"
      backendRefs:
        - name: web-service
          port: 80
```
