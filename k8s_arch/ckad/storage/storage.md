# K8S State Persistance

tags: #storage

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S State Persistance](#k8s-state-persistance)
    - [Volume](#volume)
    - [PersistentVolume](#persistentvolume)
    - [PersistentVolumeClaim](#persistentvolumeclaim)
  - [StorageClass](#storageclass)
  - [StatefulSets](#statefulsets)
  - [Headless Services](#headless-services)
  - [volumeClaimTemplates](#volumeclaimtemplates)

<!-- /code_chunk_output -->

---

**Usage:**

- statically define `PV`
  - create `PVC` pointing to `PV`
    - use `PVC` in `Pod`, `ReplicaSet` or `Deployment`
- dynamically define `PVs` w/ `StorageClass`
  - create `PVC` pointing to `StorageClass`
    - use `PVC` in `Pod`, `ReplicaSet` or `Deployment`
  - use `volumeClaimTemplates` pointing to `StorageClass` in `StatefulSets`

### Volume

[[volume]]

### PersistentVolume

[[persistentVolume]]

### PersistentVolumeClaim

[[persistentVolume]] < [persistentVolumeClaim]

## StorageClass

[[storageClass]]

## StatefulSets

[workloads] > [statefulSet]

## Headless Services

[networking] > [service] > [headless_service]

## volumeClaimTemplates

[workloads] > [statefulSet] < [volumeClaimTemplates]
