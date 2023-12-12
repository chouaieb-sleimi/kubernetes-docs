# K8S ResourceQuotas

tags: #objects #configuration

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S ResourceQuotas](#k8s-resourcequotas)

<!-- /code_chunk_output -->

---

https://kubernetes.io/docs/concepts/policy/resource-quotas/

- namespaced
- provides constraints that limit total resource consumption **per [[namespace]]**.
- can limit quantity of objects that can be created in a namespace by
  - type
  - total of compute resources that may be consumed by resources

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev

spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```