# K8S Operator Framework

tags: #objects #arch

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

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
