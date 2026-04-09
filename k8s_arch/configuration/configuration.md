# K8S Configuration

tags: #configuration

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Define, Build, Modify Container Images](#define-build-modify-container-images)
- [Commands And Arguments](#commands-and-arguments)
- [ConfigMap](#configmap)
- [Secrets](#secrets)
- [Resource Limits and Requests](#resource-limits-and-requests)
- [Taints and Tolerations](#taints-and-tolerations)
  - [Taint - Node](#taint---node)
  - [Toleration - Pod](#toleration---pod)
- [Node Selectors and Affinity](#node-selectors-and-affinity)
  - [Node Selectors](#node-selectors)
  - [Node Affinity](#node-affinity)

<!-- /code_chunk_output -->

---

## Define, Build, Modify Container Images

- `Dockerfiles`
  - commands
    - `FROM`
    - `RUN`
    - `COPY`
    - `ENTRYPOINT`
  - layers and caching
- `docker`/`podman` operations

```bash
podman build
podman history
podman push
```

## Commands And Arguments

[[cmds_args]]

## ConfigMap

[[configMap]]

## Secrets

[[secret]]

## Resource Limits and Requests

[[resource]]

## Taints and Tolerations

Are used to set **restrictions on what pods nodes CAN accept (NOT MUST)**.

### Taint - Node

[[taint]]

### Toleration - Pod

[[toleration]]

## Node Selectors and Affinity

Restrict pods to run on particular node(s), or to prefer to run on particular nodes.

### Node Selectors

[[node_selector]]

### Node Affinity

[[node_affinity]]
