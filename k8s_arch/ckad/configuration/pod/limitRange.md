# K8S LimitRange

tags: #objects #configuration #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S LimitRange](#k8s-limitrange)

<!-- /code_chunk_output -->

---

https://kubernetes.io/docs/concepts/policy/limit-range/

- namespaced
- default limit values for every [[pod]] created without requests or limits.
- affects **only newly created pods.**

create cpu LimitRange

    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-resource-constraint
    spec:
      limits:
      - type: Container
        default:          # default limit
          cpu: 500m
        defaultRequest:   # default request
          cpu: 500m
        max:              # max limit that can be set on a container
          cpu: 1
        min:              # min request a container can make
          cpu: 100m

create memory LimitRange

    apiVersion: v1
    kind: LimitRange
    metadata:
      name: memory-resource-constraint
    spec:
      limits:
      - type: Container
        default:          ## default limit
          memory: 1Gi
        defaultRequest:   # default request
          memory: 1Gi
        max:              # max limit that can be set on a container
          memory: 1Gi
        min:              # min request a container can make
          memory: 500Mi
