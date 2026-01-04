# Kubernetes Controller Manager

tags: #arch #controlplane #controller_manager

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Controllers](#controllers)
- [Manual Installation](#manual-installation)

<!-- /code_chunk_output -->

---

## Overview

- runs controller processes
- logically, each controller is a separate process
  - **to reduce complexity**, all are compiled into a **single binary** and run in a **single process**
- also performs lifecycle functions
  - namespace creation and lifecycle,
  - event garbage collection,
  - terminated-pod garbage collection,
  - cascading-deletion garbage collection,
  - node garbage collection
  - ...

See [Cloud Controller Manager for more information](https://kubernetes.io/docs/concepts/architecture/cloud-controller/).

## Controllers

see: [controllers.md](./controllers.md)

## Manual Installation

see: [[installation-manual]]
