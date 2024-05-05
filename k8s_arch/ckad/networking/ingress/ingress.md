# K8S Ingress

tags: #objects #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Ingress](#k8s-ingress)
  - [Ingress Controller](#ingress-controller)
  - [Ingress Resources](#ingress-resources)

<!-- /code_chunk_output -->

---

- exposes HTTP and HTTPS **routes from outside the cluster to services within the cluster.**
- traffic routing by rules in Ingress resource.
- provides:
  - load balancing
  - SSL termination
  - name-based virtual hosting

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

[[ingress_controller]]

## Ingress Resources

[[ingress_resource]]

