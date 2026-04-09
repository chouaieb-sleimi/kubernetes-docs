# K8S Pod Resource Limits and Requests

tags: #configuration #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Limits and Requests Overview](#limits-and-requests-overview)
- [Limits and Requests in Pods](#limits-and-requests-in-pods)

<!-- /code_chunk_output -->

---

## Limits and Requests Overview

When exceeding **CPU**, pod [[containers]] are throttled.
When exceeding **MEM**, pods are terminated to free memory and are re-created because of an **OOM**.

**CPU** must be >0.1cpu or >1m (`1cpu = 1000m`; `m: milli`)
**MEM** can be 256Mi = 268 M = 268435456

possible **requests/limits scenarios:**

- **NO REQUESTS / NO LIMITS**

  - **CPU**
    pods can consume all the resources ans starve others
  - **MEM**
    pods can consume all the resources ans starve others

- **NO REQUESTS / LIMITS**

  - **CPU**
    requests = limits
  - **MEM**
    requests = limits

- **REQUESTS / LIMITS**

  - **CPU**
    requests are guaranteed, exceeding limits results in throtelling
  - **MEM**
    requests are guaranteed, exceeding limits results in recreation

- **REQUESTS / NO LIMITS**

  - **CPU**
    exceeding limits is permitted when it doesn't starve others of requests
  - **MEM**
    exceeding limits is permitted when it doesn't starve others of requests

## Limits and Requests in Pods

set container **resource request** and **resource limits**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name: ...
          image: ...
          ports:
            - containerPort: ...
          resources:
            requests:
              memory: "4Gi"
              cpu: 2
            limits:
              memory: "6Gi"
              cpu: 3
