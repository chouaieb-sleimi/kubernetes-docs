# Kubernetes Scheduler

tags: #arch #controlplane #scheduler

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Manual Installation](#manual-installation)

<!-- /code_chunk_output -->

---

## Overview

- watches for newly created Pods with no assigned node
- selects a node for them to run on
- **scheduling factors** include:
  - individual and collective resource requirements,
  - hardware/software/policy constraints,
  - affinity and anti-affinity specifications,
  - data locality,
  - inter-workload interference,
  - deadlines.
- **stage of scheduling:**
  - filtering out ineligible nodes (resource, taints, affinity ...)
  - scoring/ranking the remaining nodes
  - selecting the highest-ranked node

## Manual Installation

see: [installation-manual.md](../../lib/installation/installation-manual.md#etcd)
