# Certified Kubernetes Application Developer - CKAD

tags: #k8s #arch

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Certified Kubernetes Application Developer - CKAD](#certified-kubernetes-application-developer---ckad)
  - [CKAD Library](#ckad-library)
  - [Section 1: Resources](#section-1-resources)
  - [Section 2: Core Concepts](#section-2-core-concepts)
    - [Docker vs ContainerD](#docker-vs-containerd)
    - [Namespaces](#namespaces)
  - [Section 3: Configuration](#section-3-configuration)
    - [Define, Build, Modify Container Images](#define-build-modify-container-images)
    - [Commands And Arguments](#commands-and-arguments)
    - [ConfigMap](#configmap)
    - [Secrets](#secrets)
    - [ServiceAccount](#serviceaccount)
    - [Resource Requirements](#resource-requirements)
      - [Resource Limits and Requests](#resource-limits-and-requests)
      - [LimitRanges](#limitranges)
    - [ResourceQuota](#resourcequota)
    - [Taints and Tolerations](#taints-and-tolerations)
    - [Node Selectors and Affinity](#node-selectors-and-affinity)
      - [Node Selectors](#node-selectors)
      - [Node Affinity](#node-affinity)
  - [Section 4: Multi-Container Pods](#section-4-multi-container-pods)
    - [Multi-Container Pods](#multi-container-pods)
    - [Init Containers](#init-containers)
  - [Section 5: Observability And API Maintenance](#section-5-observability-and-api-maintenance)
    - [Observability](#observability)
      - [Readiness and Liveness Probes](#readiness-and-liveness-probes)
        - [Pod Status](#pod-status)
        - [Pod Conditions](#pod-conditions)
      - [Readiness Probe](#readiness-probe)
      - [Liveness Probe](#liveness-probe)
      - [Container Logging](#container-logging)
      - [Monitoring Cluster](#monitoring-cluster)
        - [Metrics Server Overview](#metrics-server-overview)
    - [API Maintenance](#api-maintenance)
  - [Section 6: Pod Design](#section-6-pod-design)
    - [Labels Selectors and Annotations](#labels-selectors-and-annotations)
    - [Deployment Rollouts and Rollbacks](#deployment-rollouts-and-rollbacks)
    - [Jobs and CronJobs](#jobs-and-cronjobs)
      - [Jobs](#jobs)
      - [CronJobs](#cronjobs)
    - [Deployment Strategies](#deployment-strategies)
      - [Blue Green Deployments](#blue-green-deployments)
      - [Canary Deployments](#canary-deployments)
  - [Section 7: Services and Networking](#section-7-services-and-networking)
    - [Service](#service)
    - [Ingress](#ingress)
      - [Ingress Controller](#ingress-controller)
      - [Ingress Resources](#ingress-resources)
    - [NetworkPolicy](#networkpolicy)
    - [Port Forwarding](#port-forwarding)
  - [Section 8: State Persistance](#section-8-state-persistance)
    - [Volume](#volume)
    - [PersistentVolume](#persistentvolume)
    - [PersistentVolumeClaim](#persistentvolumeclaim)
    - [StorageClass](#storageclass)
    - [StatefulSets](#statefulsets)
    - [Headless Services](#headless-services)
    - [volumeClaimTemplates](#volumeclaimtemplates)
  - [Secition 9: Security](#secition-9-security)
    - [SecurityContext](#securitycontext)
    - [Authentication](#authentication)
    - [Authorization](#authorization)
    - [NetworkPolicy](#networkpolicy-1)
    - [Operator Framework](#operator-framework)
  - [Section 10: Helm Fundamentals](#section-10-helm-fundamentals)

<!-- /code_chunk_output -->

---

## Kubernetes Library

[[library]]

---

## Section 1: Resources

- Certified Kubernetes Application Developer: https://www.cncf.io/certification/ckad/

- Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

- Exam Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

Keep the code - 20KLOUD handy while registering for the CKA or CKAD exams at Linux Foundation to get a 20% discount.

- Kubernetes The Hard Way
  https://github.com/mmumshad/kubernetes-the-hard-way
- kubernetes io
  https://kubernetes.io
- github - Kubernetes
  https://github.com/kubernetes/kubernetes
- helm docs
  https://helm.sh/docs/

## Section 2: Core Concepts

### Docker vs ContainerD

see [[library#Docker vs ContainerD]]

### Namespaces

- object names must be unique within a namespace,
  - not across namespaces
- Namespace-based scoping is applicable only for namespaced objects,
  - not for cluster-wide objects

**Cross-namespace object name format:**
format: `<object-name>.<namespace-name>.<object-type>.<cluster-domain>`
example: `db-service.dev.service.cluster.local`

---

## Section 3: Configuration

[[configuration]]

### Define, Build, Modify Container Images

### Commands And Arguments

### ConfigMap

### Secrets

### ServiceAccount

### Resource Requirements

#### Resource Limits and Requests

#### LimitRanges

### ResourceQuota

### Taints and Tolerations

### Node Selectors and Affinity

#### Node Selectors

#### Node Affinity

---

## Section 4: Multi-Container Pods

### Multi-Container Pods

[[workloads]] > [pods] > [containers]

### Init Containers

[[workloads]] > [pods] > [containers]

---

## Section 5: Observability And API Maintenance

### Observability

[[monitoring]]

#### Readiness and Liveness Probes

##### Pod Status

##### Pod Conditions

#### Readiness Probe

#### Liveness Probe

#### Container Logging

#### Monitoring Cluster

##### Metrics Server Overview

### API Maintenance

---

## Section 6: Pod Design

### Labels Selectors and Annotations

- **annotations:** attach **arbitrary non-identifying metadata** to objects.
  - clients (ex: tools, libraries) can retrieve this metadata.
- **label selectors** can be made of multiple requirements which are comma-separated.
  - **types of selectors:**
    - **equality-based / inequality-based**
      - filtering by label keys and values.
      - objects must satisfy ALL constraints,
        - they may have additional labels
      - 3 operators supported: `=`, `==`, `!=`
    - **set-based**.
      - filtering according to a set of values.
      - 3 operators supported: `in`, `notin`, `exists`

examples

```yaml
## equality-based / inequality-based
environment = production
tier != frontend

## set-based
# value: production OR qa.
environment in (production, qa)

# value notin frontend and backend
tier notin (frontend, backend)

# key exists
partition

# key not exits
!partition

# partition AND environment keys different than qa
partition,environment notin (qa)

# equality-based and set-based selector
partition in (customerA, customerB),environment!=qa
```

### Deployment Rollouts and Rollbacks

[[workloads]] > [deployment] > [rollouts_rollbacks]

### Jobs and CronJobs

#### Jobs

[[workloads]] > [job]

#### CronJobs

[[workloads]] > [cronJob]

### Deployment Strategies

#### Blue Green Deployments

Steps:

1. create deployment v1 (blue)
2. create deployment service
3. create deployment v2 (green)
4. perform tests on v2 (optional)
5. switch service label selector to v2 (green)

#### Canary Deployments

Steps:

1. create deployment v1 (blue)
2. create deployment service
3. create deployment v2 (green)
4. route small percentage fo trafic to v2 (green)
   4a. route traffic to both versios
   use a common label
   4b. route a small % of trafic to v2
   reduce num of pods in v2 (green)
5. if no issues, switch service label selector to v2 (green)

---

## Section 7: Services and Networking

[[networking]]

### Service

### Ingress

#### Ingress Controller

#### Ingress Resources

### NetworkPolicy

### Port Forwarding

---

## Section 8: State Persistance

[[storage]]

### Volume

### PersistentVolume

### PersistentVolumeClaim

### StorageClass

### StatefulSets

### Headless Services

### volumeClaimTemplates

---

## Secition 9: Security

[[security]]

### SecurityContext

### Authentication

### Authorization

### NetworkPolicy

### Operator Framework

---

## Section 10: Helm Fundamentals

[[workloads]] > [helm]
