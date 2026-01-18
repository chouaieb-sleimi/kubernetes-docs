# K8S Volume

tags: #objects #storage

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Volume](#k8s-volume)

<!-- /code_chunk_output -->

---

see: [kubernetes.io/docs - Volumes](https://kubernetes.io/docs/concepts/storage/volumes)


- directory or file accessible to the containers in a [[pod]].
- **separate manifest not required** to create a PV
  - defined in pod manifest
- separates storage from a container, 
  - **binds storage to a Pod**.
- lifecycle of a volume is dependent on the Pod using it
- enables **safe container restart**
- allows sharing of data across containers

**volume types:** [[volume-types]]

