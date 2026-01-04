# Kubernetes Scheduler

tags: #arch #controlplane #scheduler

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Manual Installation](#manual-installation)
- [Scheduling](#scheduling)
  - [Manual Scheduling](#manual-scheduling)
  - [Priority Classes and Preemption](#priority-classes-and-preemption)
  - [Multiple Schedulers](#multiple-schedulers)
  - [Scheduling Phases/ Plugins/ Extensions](#scheduling-phases-plugins-extensions)
  - [Scheduler Profiles](#scheduler-profiles)

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

see: [[installation-manual]]

## Scheduling

### Manual Scheduling

useful for testing or specific use cases where manual control over pod placement is required.

- can manually assign a pod to a node by setting the `nodeName` field in the PodSpec.
- the scheduler **will not** attempt to schedule pods that have the `nodeName` field set.
- for already running pods, create a `Binding` object to bind the pod to a specific node.
  ```yaml
  apiVersion: v1
  kind: Binding
  metadata:
    name: my-pod
    namespace: default
  target:
    apiVersion: v1
    kind: Node
    name: my-node
  ```

### Priority Classes and Preemption

- **priority ranges:**
  - -2,147,483,647 to 1,000,000,000 for user-defined priorities
  - 1,000,000,000 to 2,000,000,000 for system priorities
  - higher value = higher priority
- **define pod priority** class: `.spec.priorityClassName: my-high-prio-class`
- **preemption:**
  - higher-priority pods can evict lower-priority pods when resources are scarce.
  - helps ensure that critical workloads get scheduled even in resource-constrained environments.
- **example definition:**
  ```yaml
  apiVersion: scheduling.k8s.io/v1
  kind: PriorityClass
  metadata:
    name: high-priority
  value: 1000000
  globalDefault: false
  preemptionPolicy: PreemptLowerPriority
  description: "This priority class is for high priority pods."
  ```

### Multiple Schedulers

- allows running multiple scheduler instances in a cluster.
- default scheduler name is `default-scheduler`.
- get pod scheduler: `kubectl get events -o wide | grep -i scheduled # REASON column`
- useful for:
  - testing new scheduling algorithms,
  - isolating workloads with different scheduling requirements,
  - implementing custom scheduling logic for specific applications.
- **use a custom scheduler:** `spec.schedulerName: my-custom-scheduler`

**basic implementation steps:**

- define `my-scheduler-config.yaml` config file

  ```yaml
  apiVersion: kubescheduler.config.k8s.io/v1beta1
  kind: KubeSchedulerConfiguration
  clientConnection:
    kubeconfig: /etc/kubernetes/my-scheduler-kubeconfig.yaml
  leaderElection: # for HA multiple scheduler instances
    leaderElect: true # false for non-active scheduler or single scheduler setups
    # lockObjectNamespace: kube-system
    # lockObjectName: my-scheduler-lock
    resourceNamespace: kube-system
    resourceName: lock-object-my-scheduler
  ```

- start my-scheduler with config file:

  ```bash
  # using scheduler binary
  /usr/local/bin/kube-scheduler --config=my-scheduler-config.yaml

  # using pod yaml file
  kubectl apply -f my-scheduler-pod.yaml
  ```

  `my-scheduler-pod.yaml`:

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-scheduler
    namespace: kube-system
  spec:
    containers:
      - name: kube-scheduler
        image: k8s.gcr.io/kube-scheduler:v1.20.0
        command:
          - kube-scheduler
          - --config=/etc/kubernetes/my-scheduler-config.yaml
        volumeMounts:
          - name: config
            mountPath: /etc/kubernetes/my-scheduler-config.yaml
            subPath: my-scheduler-config.yaml
    volumes:
      - name: config
        configMap:
          name: my-scheduler-config
  ```

see docs: [Using Multiple Schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

### Scheduling Phases/ Plugins/ Extensions

1. **Scheduling Queue:**
   - validates pod priority and places it in the scheduling queue.
    - high priority pods are scheduled first.
   - extension: `QueueSort`
   - plugin: `PrioritySort`
2. **Filtering:**
   - filters out nodes that do not meet the pod's requirements.
   - extensions: `Filter`, `PreFilter`, `PostFilter`
   - plugins: `NodeResourcesFit`, `NodeAffinity`, `NodeUnschedulable`, `TaintToleration`, `NodePorts`, `NodeName`
3. **Scoring:**
   - scores the remaining nodes based on various criteria.
   - extensions: `Score`, `PreScore`, `Reserve`
   - plugins: `NodeResourcesFit`, `ImageLocality`, `NodeResourcesBalancedAllocation`, `InterPodAffinity`
4. **Binding:**
   - binds the pod to the selected node.
   - extensions: `Bind`, `PostBind`, `preBind`, `Permit`
   - plugin: `DefaultBinder`
  
see docs: [Kubernetes Scheduler Extensibility](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/)

### Scheduler Profiles

- allow configuring multiple scheduling profiles within a single scheduler instance (binary).
- each profile can have its own set of scheduling policies, plugins, and configurations.
- pods can specify which profile to use via the `schedulerName` field in the PodSpec.
- useful for isolating different workloads with distinct scheduling requirements within the same cluster.
- example config with multiple profiles:
  ```yaml
  apiVersion: kubescheduler.config.k8s.io/v1beta1
  kind: KubeSchedulerConfiguration
  profiles:
    - schedulerName: my-scheduler-2
      plugins:
        filter:
          disabled:
            - name: NodeUnschedulable
          enabled:
            - name: NodeResourcesFit
            - name: myCustomPluginA
            - name: myCustomPluginB
        score:
          enabled:
            - name: NodeResourcesBalancedAllocation
    - schedulerName: my-scheduler-3
      plugins:
        preScore:
          enabled:
            - name: '*'
        score:
          enabled:
            - name: '*'
  ```
