# K8S Containers

tags: #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Containers](#k8s-containers)
  - [Multi-Container Pods](#multi-container-pods)
    - [Init Containers](#init-containers)
  - [Interfaces](#interfaces)

<!-- /code_chunk_output -->

---

## Multi-Container Pods

- for tightly coupled containers that need to share resources.
- containers share
  - lifecycle,
  - network space (they can reference each other with `localhost`)
  - storage volumes.

Multi-Container Pods Design Patterns

- **[[sidecar]]**
  The sidecar pattern consists of a main application + a helper container with a responsibility that is essential to your application, but **is not necessarily part of the application itself.**

- **[[adapter]]**
  The adapter pattern is used to **standardize and normalize application output or monitoring data for aggregation.**

- **[[ambassador]]**
  The ambassador pattern is a useful way to **connect containers with the outside world.**
  An ambassador container is essentially a proxy that allows other containers to connect to a port on localhost while the ambassador container can proxy these connections to different environments depending on the cluster's needs.

### Init Containers

[[init_container]] is used to run a process that runs to completion in a container.

can be:

- a task that will be **run only one time when the pod is first created**
- a process that waits for an external service or database to be up before the actual application starts

**define an initContainer** in a pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:

      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']

      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']

- `initContainers` processs must run to a completion before `containers` starts.
- multiple `initContainers` run one at a time in sequential order.

If one of the `initContainers` fail to complete, Pod restarts repeatedly until the `initContainers` succeeds.

`initContainers` example

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:

      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']

      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']

      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']

Read more about initContainers here.
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

## Interfaces

non-kubernetes (MesOS, CloudFoundry,...) universal standards that allow orchestration tools to work w/ 3rd party vendor

- **Container Runtime Interfaces (CRI)**
  - rkt
  - containerd
  - cri-o
- **Container Storage Interfaces (CSI)**
  see [[container-storage]]
  - calico
  - flannel
  - cilium
- **Container Network Interfaces (CNI)**
  - portworks
  - Amazon EBS
  - DellEMC
  - Azure Disk
  - GlusterFS
  - ...
