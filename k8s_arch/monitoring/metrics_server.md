# K8S Metrics Server

tags: #objects #monitoring

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Metrics Server Overview](#metrics-server-overview)
- [Metrics Server Deployment](#metrics-server-deployment)

<!-- /code_chunk_output -->

---

### Metrics Server Overview

[[metrics_server]]

Is a slimmed down version of heapster. can only be 1 metrics server per k8s cluster. An **In-Memory monitoring solution**; doesn't store logs and mterics data on disk.

Uses a `kubelet` component `cAdvisor`; retrieves pod performance metrics and expose them through kubelet api to metrics server.

### Metrics Server Deployment

deploy metrics server in minikube as an addon

```bash
minikube addons enable metrics-server
```

kubeadm docs: https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/


deploy metrics server in other clusters

```bash
# clone yaml deployment files
# contain set of pods, services and roles
git clone https://github.com/kubernetes-incubator/metrics-server.git

# deploy metrics server
kubectl create -f deploy/1.8+/
```

get performance metrics

```bash
kubectl top node
kubectl top pod
```
