# Kubernetes Maintenance

tags: #arch #maintenance

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [OS Patching](#os-patching)
- [Cluster Upgrade](#cluster-upgrade)
  - [Kubernetes Releases](#kubernetes-releases)
  - [Cluster Upgrade Process](#cluster-upgrade-process)
- [Backup & Restore](#backup--restore)

<!-- /code_chunk_output -->

---

## OS Patching

[[os-patching]]

1. **node drain:** move pods to other nodes (cordon + evict)
   - evict: remove pods from node
     - graceful termination of pods
     - recreate pods on other nodes (if managed by controller)
   - **cordon:** mark node as unschedulable
2. apply OS patches & reboot
3. **node uncordon:** mark node as schedulable again

## Cluster Upgrade

[[cluster-upgrade]]

### Kubernetes Releases

- **version format:** `major.minor.patch-[alpha|beta]` (`v1.33.0`)
  - major: incompatible API changes
  - minor: new features, backward-compatible
  - patch: backward-compatible bug fixes
- **release types:** stable, alpha, beta
  - alpha: experimental features, new features are not enabled by default
  - beta: experimental features, enabled by default
- **packaging:** released as binaries, container images, and packages for various OS
- **control-plane version hierarchy** `(x = version x.y.z)`:
  1. `kube-apiserver(x)`
  2. `kube-controller-manager(x-1)`
     `kube-scheduler(x-1)`
  3. `kubelet(x-2)`
     `kube-proxy(x-2)`
  - `kubectl(x-1 to x+1)`

> note: control plane components follow the same versioning scheme except for etcd, CoreDNS which have their own versioning

see:

- kubernetes docs api-versioning: https://kubernetes.io/docs/reference/using-api/api-overview/#api-versioning
- github api-conventions: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
- github api_changes: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-changes.md

### Cluster Upgrade Process

- **release cycle:** ~3 months
- **support policy:** 3 most recent minor versions
  - e.g., if current version is v1.33.x, supported versions are v1.33.x, v1.32.x, v1.31.x
- **upgrade strategy:**
  1. upgrade control plane first
  2. then upgrade worker nodes
- **upgrade tools:**
  - manual upgrade (hard, not recommended)
  - kubeadm
    - doesn't upgrade kubelet because kubelet doesn't run in a pod
  - managed k8S services (EKS, GKE, AKS)
- **best practices:**
  - test upgrades in staging environment
  - read release notes for breaking changes
  - backup etcd before upgrade

see: kubeadm upgrade docs - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

## Backup & Restore

**references:**

- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd#backing-up-an-etcd-cluster
- https://github.com/etcd-io/etcd/blob/main/Documentation/op-guide/recovery.md
- Disaster Recovery for your Kubernetes Clusters
  https://www.youtube.com/watch?v=qRPNuT080Hk

**backup targets:**

- cluster state (resource manifests, configs)
  tools: `kubectl`, `velero`, `kube-backup`, etc.
- etcd data
  tools: `etcdctl`, `restic`, etc.
- persistent data (volumes)
  tools: `velero`, `restic`, etc.
