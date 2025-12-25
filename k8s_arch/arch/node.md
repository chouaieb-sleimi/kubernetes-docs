# K8S Worker Nodes

tags: #arch #dataplane

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Worker Nodes](#k8s-worker-nodes)
  - [Components](#components)
    - [kubelet](#kubelet)
    - [kube-proxy](#kube-proxy)
    - [container-runtime](#container-runtime)

<!-- /code_chunk_output -->

---

## Components

### kubelet

see: [kubelet.md](./data-plane/kubelet.md)

### kube-proxy

see: [kube-proxy.md](./data-plane/kube-proxy.md)

### container-runtime

see: [container-runtime.md](./data-plane/container-runtime.md)

- software that is responsible for running containers
- examples of container runtimes:
  - **containerd**,
  - ~~**docker**~~ (deprecated),
  - **rkt**,
  - **cri-o**,
  - any other implementation of k8s CRI (Container Runtime Interface).
    - **cri-o**
    - **cri-containerd**
    - ...
