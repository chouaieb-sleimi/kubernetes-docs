# K8S Operator Framework

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Operator Framework](#k8s-operator-framework)
    - [CustomControllers](#customcontrollers)
    - [CustomResourceDefinition](#customresourcedefinition)

<!-- /code_chunk_output -->

---

see: [kubernetes.io - An example operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/#example)

**Community operators:** https://operatorhub.io

**Operators** = **Custom Controllers** + **Custom Resource Definitions (CRDs)**

example

    Custom Resource Definition (CRD)      Custom Controller
    ------------------------------------------------------------
    EtcdCluster                           ETCD Controller
    EtcdBackup                            Backup Operator
    EtcdRestore                           Restore Operator

### CustomControllers

[[customController]]

### CustomResourceDefinition

[[customResourceDefinition]]
