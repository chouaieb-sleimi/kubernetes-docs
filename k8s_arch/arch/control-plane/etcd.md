# Kubernetes - ETCD

tags: #arch #controlplane #etcd

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Kubernetes - ETCD](#kubernetes---etcd)
  - [Overview](#overview)
  - [Manual Installation](#manual-installation)

<!-- /code_chunk_output -->

---

## Overview

- consistent and highly-available key value store
- used as k8S backing store for all cluster data
- all cluster states are stored here
- version 3 is used by k8S
  - provides a watch mechanism to get notified of changes

## Manual Installation

see: [installation-manual.md](../../lib/installation/installation-manual.md#etcd)
