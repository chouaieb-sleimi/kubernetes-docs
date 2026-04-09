# K8s Static Pods

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->

---

- **Static Pods** are **managed directly** by the kubelet on a specific node.
  - are typically **used for critical system components** that need to run on specific nodes, such as the control plane components in a single-node cluster.
  - can be useful for **bootstrapping a cluster** or **running essential services** that must be available even if the API server is down.
- **are ignored by the scheduler**.
- **are not managed by the API server**, they do not appear in the list of pods when using `kubectl get pods` unless the API server is aware of them through other means (e.g , by creating a corresponding Pod object).
  - in a cluster setup, **kubelet can create a mirror Pod object** in the API server to represent the static pod, allowing it to be visible through the API.
- defined by placing a Pod manifest file in a specific directory on the node (usually `/etc/kubernetes/manifests`).

  - path can be configured:
    - using the `--pod-manifest-path` flag when starting the kubelet from commandline.
    - via `staticPodPath` field in the kubelet config file passed in the `--config=<config_file>`.
  - kubelet monitors this directory and automatically creates, updates or deletes the static pods defined there.

**useful commands:**

  ```bash
  # cri-o
  crictl ps | grep static-pod-name   # list static pod containers
  crictl logs <container-id>         # view logs of a static pod container

  # containerd
  ctr -n k8s.io containers list | grep static-pod-name   # list static pod containers
  nerdctl -n k8s.io containers list | grep static-pod-name   # list static pod containers

  ctr -n k8s.io tasks logs <container-id>                 # view logs of a static pod container
  nerdctl -n k8s.io logs <container-id>                     # view logs of a static pod container
  ```
