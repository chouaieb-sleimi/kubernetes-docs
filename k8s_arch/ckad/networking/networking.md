# K8S Networking

tags: #objects #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Networking](#k8s-networking)
  - [Services](#services)
    - [NodePort](#nodeport)
    - [ClusterIP](#clusterip)
    - [LoadBalancer](#loadbalancer)
  - [Ingress](#ingress)
    - [Ingress Controller](#ingress-controller)
    - [Ingress Resources](#ingress-resources)
  - [NetworkPolicy](#networkpolicy)
    - [Port Forwarding](#port-forwarding)

<!-- /code_chunk_output -->

---

- Internal Private Network is created when k8s is created/configured
- Each pod has an ip address

**K8S Fundamental Networking Requirements:**

- **Containers or PODs** in a cluster MUST be able to **communicate without configuring NAT**.

- **All nodes** must be able to **communicate with containers** in the cluster.
- **All containers** must be able to **communicate with the nodes** in the cluster.

## Services

[[service]]

### NodePort

[[service]] > nodePort

### ClusterIP

[[service]] > clusterIP

### LoadBalancer

[[service]] > loadBalancer

## Ingress

[[ingress]]

### Ingress Controller

[[ingress]] > [ingress_controller]

### Ingress Resources

[[ingress]] > [ingress_resource]

## NetworkPolicy

[[networkPolicy]]

### Port Forwarding

[[port_forwarding]]

