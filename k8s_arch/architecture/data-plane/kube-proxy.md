# Kubernetes Kube-Proxy

tags: #arch #dataplane #kube-proxy

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [overview](#overview)
- [Manual Installation](#manual-installation)

<!-- /code_chunk_output -->

---

## overview

- process that runs on each node in cluster
- watches the API server for changes to Service and Endpoint objects
- **maintains network rules** on nodes.
  - network rules allow network communication to Pods from network sessions inside or outside the cluster
  - uses **os packet filtering layer if there is one** and it's available
  - Otherwise, **forwards the traffic itself**
- supports service abstraction (ClusterIP, NodePort, LoadBalancer)
- supports different proxy modes:
  - **iptables** (default, recommended)
    - service: db(10.96.0.12) 
    -> kube-proxy on nodeA > podIP(10.32.0.14)
    -> kube-proxy on nodeB > podIP(10.32.0.15)
  - **ipvs** (more performant, requires additional setup)
- can be run in userspace mode (deprecated)

## Manual Installation

see: [[installation-manual]]
