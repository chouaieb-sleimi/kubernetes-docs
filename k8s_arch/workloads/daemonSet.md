# K8S DaemonSet

tags: #objects #workloads

---

- Creates and manages a [[replicaSet]].
- will create a pod in each node of the cluster.
- managed by the kube-controller-manager (DaemonSet Controller)
- ignored by the scheduler.
- used for deploying **daemon applications** such as:
  - log collectors,
  - monitoring agents,
  - networking plugins,
  - storage plugins,
  - etc.
