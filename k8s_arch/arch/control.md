# K8S Control Plane

tags: #arch

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Control Plane](#k8s-control-plane)
  - [Components](#components)
    - [Controllers](#controllers)

<!-- /code_chunk_output -->

---

- control plane components doc:
  https://kubernetes.io/docs/concepts/overview/components/#control-plane-components

## Components

- **[[api-server]]**
  - exposes the k8S API
  - front end for the k8S control plane
  - designed to scale horizontally
    - can run several instances of apiserver and balance traffic between then
- **[[etcd]]**
- **[[scheduler]]**

  - watches for newly created Pods with no assigned node
  - selects a node for them to run on
  - **scheduling factors** include:
    - individual and collective resource requirements,
    - hardware/software/policy constraints,
    - affinity and anti-affinity specifications,
    - data locality,
    - inter-workload interference,
    - deadlines.

- **[[controller-manager]]**
  - runs controller processes
  - logically, each controller is a separate process
    - **to reduce complexity**, all are compiled into a **single binary** and run in a **single process**
  - also performs lifecycle functions
    - namespace creation and lifecycle,
    - event garbage collection,
    - terminated-pod garbage collection,
    - cascading-deletion garbage collection,
    - node garbage collection
    - ...

### Controllers

- loops that watch the state of your cluster (API server)
  - tracks **at least** one Kubernetes resource type
    - these objects have a spec field that represents the **desired state**
- make or request changes where needed
- use the **watch mechanism** to get notified of changes

See [Cloud Controller Manager for more information](https://kubernetes.io/docs/concepts/architecture/cloud-controller/).
