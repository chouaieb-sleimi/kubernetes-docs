# K8S Control Plane

tags: #arch #controlplane

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Control Plane](#k8s-control-plane)
  - [Components](#components)
    - [API-Server](#api-serverapi-servermd)
    - [ETCD](#etcdetcdmd)
    - [Scheduler](#schedulerschedulermd)
    - [Controller-Manager](#controller-managercontroller-managermd)
    - [Controllers](#controllers)

<!-- /code_chunk_output -->

---

- control plane components doc:
  https://kubernetes.io/docs/concepts/overview/components/#control-plane-components

## Components

can be run in containers or directly on the host machine

### [API-Server](api-server.md)
  - exposes the k8S API
  - front end for the k8S control plane
  - designed to scale horizontally
    - can run several instances of apiserver and balance traffic between then
### [ETCD](etcd.md)
  - consistent and highly-available key value store
  - used as k8S backing store for all cluster data
  - all cluster states are stored here
  - version 3 is used by k8S
    - provides a watch mechanism to get notified of changes
### [Scheduler](scheduler.md)
  - watches for newly created Pods with no assigned node
  - selects a node for them to run on
  - **scheduling factors** include:
    - individual and collective resource requirements,
    - hardware/software/policy constraints,
    - affinity and anti-affinity specifications,
    - data locality,
    - inter-workload interference,
    - deadlines.

### [Controller-Manager](controller-manager.md)
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

- **[[container-runtime]]**
  see [node.md#components](node.md#components) > container-runtime

### Controllers

- loops that watch the state of your cluster (API server)
  - tracks **at least** one Kubernetes resource type
    - these objects have a spec field that represents the **desired state**
- make or request changes where needed
- use the **watch mechanism** to get notified of changes

See [Cloud Controller Manager for more information](https://kubernetes.io/docs/concepts/architecture/cloud-controller/).
