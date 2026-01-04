# K8S Deployment

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Deployment](#k8s-deployment)
  - [Use Cases](#use-cases)
  - [Rollout Strategies](#rollout-strategies)
  - [Rollouts, Rollbacks](#rollouts-rollbacks)

<!-- /code_chunk_output -->

---

## Use Cases

- **rollout** a [[replicaSet]]
- **update** the PodTemplateSpec. new [[replicaSet]] is created. new [[replicaSet]] updates the revision of the Deployment.
- **rollback** to an earlier Deployment revision. each rollback updates the revision.
- **scale-up** pods.
- **pause** the rollout of a Deployment
  - to apply fixes to PodTemplateSpec and then resume it to _start a new rollout_.
- **monitor** rollout status.
- **clean-up** older replicaSets.

## Rollout Strategies

- **recreate** destroys all old app version instances **then** launch new one
- **rolling update** -*default strategy*- recreate instances **few-at-a-time**

> Note: adding the `--record` option to _deployment creation and modification commands_ allows the recording of change cause in the revision history

## Rollouts, Rollbacks

[[rollout_rollback]]