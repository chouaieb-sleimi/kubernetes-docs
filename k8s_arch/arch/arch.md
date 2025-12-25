# K8S Architecture

tags: #arch

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Architecture](#k8s-architecture)
  - [References](#references)
  - [Documentation](#documentation)
  - [K8S Components](#k8s-components)
    - [Control Plane](#control-plane)
    - [Nodes](#nodes)
    - [Namespace](#namespace)

<!-- /code_chunk_output -->

---

## References

- [kubernetes.io](https://kubernetes.io)
- [github.com - Kubernetes](https://github.com/kubernetes/kubernetes)
- [helm docs](https://helm.sh/docs/)

## Documentation

- [kubernetes.io - Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [opensource.com - A guide to Kubernetes architecture](https://opensource.com/article/22/2/kubernetes-architecture)
- [opensource.com - A visual guide to Kubernetes networking fundamentals](https://opensource.com/article/22/6/kubernetes-networking-fundamentals?utm_medium=Email&utm_campaign=weekly&sc_cid=7013a00000311fXAAQ)
- [opensource.com - A visual map of a Kubernetes deployment](https://opensource.com/article/22/3/visual-map-kubernetes-deployment)
- [redhat.com - How Kubernetes creates and runs containers: An illustrated guide](https://www.redhat.com/architect/how-kubernetes-creates-runs-containers)
- [medium.com - Scaling Kubernetes to Over 4k Nodes and 200k Pods](https://medium.com/paypal-tech/scaling-kubernetes-to-over-4k-nodes-and-200k-pods-29988fad6ed)

---

## K8S Components

### Control Plane

see [control.md](control.md)

**components:**

  - api-server
  - etcd
  - scheduler
  - controller manager

### Nodes

see [node.md](node.md)

**components:**

- kubelet
- kube-proxy
- container runtime

### Namespace

see [namespace.md](namespace.md)

**components:**

- resource quota
- network policy
- limit range
