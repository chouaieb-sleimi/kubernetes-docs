# K8S ReplicationController

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->

---

- ensures that a specified number of [[pod]] replicas are running at any one time.
- can manage pods not started by it
- `selector` field is not required,
  - if skipped it assumes the same labels in the pod definition
