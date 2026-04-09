# K8S ReplicaSet

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->

---

- can manage pods not started by it
- is linked to its Pods via the [[pod]] `metadata.ownerReferences` field,

  - specifies what resource the current object is owned by.
  - Pods acquired by a RS have their owning RS's identifying information within their `ownerReferences`
  - through this link the RS knows of the state of the Pods it is maintaining and plans accordingly.

- identifies new Pods to acquire by using its `selector`.
- If there is a Pod

  - has no `OwnerReference`
  - or the `OwnerReference` is not a ` `
  - matches a ReplicaSet's selector,
  - it will be immediately acquired by said ReplicaSet.

- When a replicaset is deleted via `kubectl delete` its pods are also deleted
