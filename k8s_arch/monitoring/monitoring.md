# K8S Observability & Monitoring

tags: #objects #monitoring

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Readiness and Liveness Probes](#readiness-and-liveness-probes)
  - [Pod Status](#pod-status)
  - [Pod Conditions](#pod-conditions)
- [Readiness Probe](#readiness-probe)
- [Liveness Probe](#liveness-probe)
- [Container Logging](#container-logging)
- [Monitoring Cluster](#monitoring-cluster)
  - [Metrics Server Overview](#metrics-server-overview)

<!-- /code_chunk_output -->

---

- types of probes
  https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#types-of-probe

## Readiness and Liveness Probes

- failing **liveness probe:** restart container
- failing **readiness probe:** stop container from serving traffic

### Pod Status

**Pod States:**

- Pending (Pod not scheduled)
- ContainerCreating
- Running

get pod status

    kubectl get pods
    kubectl describe pod <pod-name> | grep -i status

### Pod Conditions

Pod conditions compliment pod status. **true or false.**

**Pod conditions:**

- PodScheduled
- Initialized
- ContainerReady (containers are running)
- Ready (pod is running)

get pod conditions

    kubectl describe pod <pod-name> | grep -iA5 conditions

## Readiness Probe

[[readiness_probe]]

## Liveness Probe

[[liveness_probe]]

## Container Logging

get a container's logs

    # <container-name> is necessary for multi-container pods.
    kubectl logs -f <pod-name> [<container-name>]

## Monitoring Cluster

**metrics:**

- **node-level metrics**

  - node number
  - node health
  - node performance metrics
    - cpu
    - memory
    - network
    - disk

- **pod-level metrics**
  - number of pods
  - pods performance metrics
    - cpu
    - memory


other K8s **monitoring solutions:**

- heapster **(deprecated)**
- metric server
- prometheus
- ELK stack
- data dog (proprietary)
- dynatracee (proprietary)

### Metrics Server Overview

[[metrics_server]]
