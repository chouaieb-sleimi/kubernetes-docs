# Kubernetes Kubelet

tags: #arch #dataplane #kubelet

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Manual Installation](#manual-installation)

<!-- /code_chunk_output -->

---

## Overview

- registered with the API server as a node
- agent that runs on each node in the cluster
- ensures that containers described in **PodSpecs** are running and healthy.
  - interacts with the container runtime to start, stop, and manage containers
- doesn't manage containers which were not created by k8S

## Manual Installation

see: [installation-manual.md](../../lib/installation/installation-manual.md#etcd)
