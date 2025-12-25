# Kubernetes API-Server

tags: #arch #controlplane #apiserver

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Kubernetes API-Server](#kubernetes-api-server)
  - [Overview](#overview)
  - [Manual Installation](#manual-installation)

<!-- /code_chunk_output -->

## Overview

- exposes the k8S API
- front end for the k8S control plane
- designed to scale horizontally
  - can run several instances of apiserver and balance traffic between then
- functions:
  - expose the REST API
    - validate requests & objects
    - authenticate users
    - authorize requests
  - retrieve data
  - update etcd

kubeadm apiserver options path (pod definition file):
`/etc/kubernetes/manifests/kube-apiserver.yaml`

## Manual Installation

see: [installation-manual.md](../../lib/installation/installation-manual.md#api-server)
