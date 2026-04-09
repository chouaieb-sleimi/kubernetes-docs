# Kubernetes - ETCD

tags: #arch #controlplane #etcd

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Manual Installation](#manual-installation)
- [ETCD in HA](#etcd-in-ha)

<!-- /code_chunk_output -->

---

## Overview

- consistent and highly-available key value store
- used as k8S backing store for all cluster data
- all cluster states are stored here
- version 3 is used by k8S
  - provides a watch mechanism to get notified of changes

## Manual Installation

see: [[installation-manual]]

## ETCD in HA

can be run as a clustered service
**leader/follower architecture**

- read from all nodes
- writes only made by the leader
  - followers forward writes to leader
  - leader forwards written data to followers - write considered complete if done on quorum of cluster members

**leader election:** RAFT protocol

- **election process**
  - on each memeber start: random timers start
  - on timer end: member sends request to become leader
  - other members give vote to requester
  - after election, leader sends notifs to other members to coontinue being leader
- in case of leader notifs stop (leader going down?) - rest of members re-initiate election process using quorum

**quorum (majority)** = N/2+1

- fault tolerance doesn't change for even number cluster size
  - example: cases of network segmentation
- recommended odd number of nodes

| instances | quorum | fault-tolerance |
| --------- | ------ | --------------- |
| 1         | 1      | 0               |
| 2         | 2      | 0               |
| 3         | 2      | 1               |
| 4         | 3      | 1               |
| 5         | 3      | 2               |
| 6         | 4      | 2               |
| 7         | 4      | 3               |


