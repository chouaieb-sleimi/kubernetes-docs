# K8S Worker Nodes

tags: #arch

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Worker Nodes](#k8s-worker-nodes)
  - [Components](#components)

<!-- /code_chunk_output -->

---

## Components

- **[[kubelet]]**
  - agent that runs on each node in the cluster
  - ensures that **containers are running in a Pod**
  - ensures that containers described in **PodSpecs** are running and healthy.
  - doesn't manage containers which were not created by k8S
- **[[kube-proxy]]**

  - network proxy that runs on each node in cluster
  - **maintains network rules** on nodes.
    - network rules allow network communication to Pods from network sessions inside or outside the cluster
    - uses **os packet filtering layer if there is one** and it's available
    - Otherwise, **forwards the traffic itself**

- **[[container-runtime]]**
  - responsible for managing the execution and lifecycle of containers
  - supported container runtimes:
    - **containerd**,
    - **CRI-O**,
    - any other **implementation of k8s CRI** (Container Runtime Interface).
      - **cri-o**
      - **cri-containerd**
      - ...
