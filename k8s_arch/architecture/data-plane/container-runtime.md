# Kubernetes Container Runtime

tags: #arch #dataplane #container-runtime

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)

<!-- /code_chunk_output -->

---

## Overview

- software that is responsible for running containers
- examples of container runtimes:
  - **containerd**,
  - ~~**docker**~~ (deprecated),
  - **rkt**,
  - **cri-o**,
  - any other implementation of k8s CRI (Container Runtime Interface).
    - **cri-o**
    - **cri-containerd**
    - ...
