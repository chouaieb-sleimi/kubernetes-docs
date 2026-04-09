# K8S Control Plane

tags: #arch #controlplane

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Components](#components)
  - [Required Components](#required-components)
    - [API-Server](#api-server)
    - [ETCD](#etcd)
    - [Scheduler](#scheduler)
    - [Controller-Manager](#controller-manager)
  - [Optional Components](#optional-components)
    - [kubelet](#kubelet)
    - [kube-proxy](#kube-proxy)
    - [container-runtime](#container-runtime)
- [Addons / Other Components](#addons--other-components)

<!-- /code_chunk_output -->

---

## Components

can be run in containers or directly on the host machine
**documentation:** https://kubernetes.io/docs/concepts/overview/components/#control-plane-components

### Required Components

#### API-Server

see: [api-server.md](./control-plane/api-server.md)

#### ETCD

see: [etcd.md](./control-plane/etcd.md)

#### Scheduler

see: [scheduler.md](./control-plane/scheduler.md)

#### Controller-Manager

see: [controller-manager.md](./control-plane/controller-manager.md)
### Optional Components

allows scheduling and running workloads; not recommended for production

#### kubelet

see [[data-plane#kubelet]] > kubelet

#### kube-proxy

see [[data-plane#kube-proxy]] > kube-proxy

#### container-runtime

see [[data-plane#container-runtime]] > container-runtime

---

## Addons / Other Components

- **cloud-controller-manager**

  Runs cloud-specific controllers that interact with the cloud API; usually deployed by cloud integrations.

  - **Type:** Control-plane component (optional)
  - **Origin:** Upstream Kubernetes component (cloud-provider specific)

- **dns (CoreDNS)**

  Provides service discovery and DNS for pods/services; required for normal cluster name resolution.

  - **Type:** Add-on (cluster DNS)
  - **Origin:** Upstream/default add-on (CoreDNS is the Kubernetes default)

- **dashboard**

  Web UI for cluster management and troubleshooting; not required for cluster operation.

  - **Type:** Add-on (optional)
  - **Origin:** Community-maintained project (kubernetes-dashboard)

- **monitoring**

  Collects metrics and alerts; recommended for production but not part of the core control plane.

  - **Type:** Add-on (observability stack)
  - **Origin:** Generally third‑party/community projects (Prometheus, Grafana, etc.)

- **logging**

  Aggregates, stores and queries logs; optional and implemented via external components.

  - **Type:** Add-on (cluster logging)
  - **Origin:** Third‑party/community projects (Fluentd/Fluent Bit, Elasticsearch, Loki, etc.)

- **cluster-autoscaler**

  Automatically scales node groups based on pod scheduling needs; runs as a controller in the cluster.

  - **Type:** Add-on (controller)
  - **Origin:** Upstream/community project (integrates with cloud providers)

- **network add-ons (CNI plugins)**

  Provide pod networking, policies, and overlays; a CNI implementation must be installed for a functional cluster.

  - **Type:** Add-on (required for pod networking)
  - **Origin:** Third‑party/community projects (Calico, Flannel, Weave, Cilium, etc.)
