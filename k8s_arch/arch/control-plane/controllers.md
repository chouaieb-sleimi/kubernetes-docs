# Kubernetes Controllers

tags: #arch #controlplane #controllers

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Controllers](#controllers)

<!-- /code_chunk_output -->

---

## Overview

- processes/loops that watch the state of your cluster (API server)
  - tracks **at least** one Kubernetes resource type
    - these objects have a spec field that represents the **desired state**
- make or request changes where needed
- use the **watch mechanism** to get notified of changes
- packages controllers into a single binary to reduce complexity (kube-controller-manager)

## Controllers

**Full controller list** and details: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/

**Main controllers list:**

- **[[Node Controller]]:** responsible for noticing and responding when nodes go down
- **[[Namespace Controller]]:** manages namespace creation and deletion
- **[[PV-binder Controller]]:** binds PersistentVolumes to PersistentVolumeClaims
- **[[PV-Protection Controller]]:** marks PersistentVolumes as "released" when their claim is deleted
- **[[PV-Reclaim Controller]]:** cleans up PersistentVolumes after their release
- **[[Service Account & Token Controllers]]:** create default accounts and API access tokens for new namespaces

- **[[Deployment Controller]]:** provides declarative updates for Pods and ReplicaSets
- **[[DaemonSet Controller]]:** ensures that all (or some) Nodes run a copy of a Pod
- **[[StatefulSet Controller]]:** manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods
- **[[Replication Controller]]:** maintains the correct number of pods for every replication controller object in the system

- **[[Endpoints Controller]]:** populates the Endpoints object (that is, joins Services & Pods)
- **[[Garbage Collector Controller]]:** cleans up resources that have been marked for deletion
- **[[Horizontal Pod Autoscaler Controller]]:** automatically scales the number of pods in a replication controller, deployment, or replica set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics)

- **[[CronJob Controller]]:** creates Jobs on a repeating schedule
- **[[Job Controller]]:** creates one or more Pods and ensures that a specified number of them successfully terminate
