# Certified Kubernetes Administrator - CKA

tags: #k8s

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Certified Kubernetes Administrator - CKA](#certified-kubernetes-administrator---cka)
  - [Kubernetes Library](#kubernetes-library)
  - [Section 1: Resources](#section-1-resources)
  - [Section 2: Core Concepts](#section-2-core-concepts)
    - [Cluster Architecture](#cluster-architecture)
    - [Docker vs ContainerD](#docker-vs-containerd)
    - [Control Plane Components](#control-plane-components)
    - [Data Plane Components](#data-plane-components)
  - [Section 3: Scheduling](#section-3-scheduling)
    - [Manual Scheduling](#manual-scheduling)
    - [Labels and Selectors](#labels-and-selectors)
    - [Taints and Tolerations](#taints-and-tolerations)
    - [Node Selectors](#node-selectors)
    - [Node Affinity](#node-affinity)
    - [Resource Limits](#resource-limits)
    - [DeamonSets](#deamonsets)
    - [Static Pods](#static-pods)
    - [Priority Classes](#priority-classes)
    - [Multiple Schedulers](#multiple-schedulers)
    - [Scheduler Profiles](#scheduler-profiles)
    - [Admission Controllers](#admission-controllers)
      - [Custom/Dynamic Admission Controllers](#customdynamic-admission-controllers)
  - [Section 4: Logging and Monitoring](#section-4-logging-and-monitoring)
  - [Section 5: Application Lifecycle Management](#section-5-application-lifecycle-management)
    - [Rolling Updates and Rollbacks](#rolling-updates-and-rollbacks)
    - [Configure Applications](#configure-applications)
    - [Commands and Arguments](#commands-and-arguments)
    - [Environment Variables](#environment-variables)
    - [Secrets](#secrets)
    - [Scaling](#scaling)
      - [Manual Scaling](#manual-scaling)
      - [Dynamic Scaling - Horizontal](#dynamic-scaling---horizontal)
      - [Dynamic Scaling - Vertical](#dynamic-scaling---vertical)
  - [Section 6: Cluster Maintenance](#section-6-cluster-maintenance)
    - [OS Patching](#os-patching)
    - [Kubernetes Releases](#kubernetes-releases)
    - [Cluster Upgrade](#cluster-upgrade)
    - [Backup & Restore](#backup--restore)
  - [Section 7: Security](#section-7-security)
    - [Basic Authentication (Passwords and Tokens)](#basic-authentication-passwords-and-tokens)
    - [TLS Basics](#tls-basics)
    - [TLS in Kubernetes](#tls-in-kubernetes)
    - [View Certificate Details](#view-certificate-details)
    - [Certificates API](#certificates-api)
    - [KubeConfig](#kubeconfig)
    - [API Groups](#api-groups)
    - [Authorization](#authorization)
      - [RBAC](#rbac)
    - [ServiceAccount](#serviceaccount)
    - [Image Security](#image-security)
    - [Container Security](#container-security)
    - [Network Security](#network-security)
  - [Section 8: Storage](#section-8-storage)
    - [Docker/Container Storage](#dockercontainer-storage)
    - [Container Runtime Interface (CRI)](#container-runtime-interface-cri)
    - [Volumes](#volumes)
    - [PersistentVolumes](#persistentvolumes)
    - [PersistentVolumeClaims](#persistentvolumeclaims)
    - [StorageClasses](#storageclasses)
  - [Section 9: Networking](#section-9-networking)
    - [Intro to Switching Routing Gateways](#intro-to-switching-routing-gateways)
    - [Intro to DNS](#intro-to-dns)
    - [CoreDNS](#coredns)
    - [Network Namespaces](#network-namespaces)
    - [Docker Networking](#docker-networking)
    - [Container Networking Interface (CNI)](#container-networking-interface-cni)
    - [Cluster Networking](#cluster-networking)
    - [Pod Networking](#pod-networking)
    - [Weave CNI + IPAM](#weave-cni--ipam)
    - [Service Networking](#service-networking)

<!-- /code_chunk_output -->

---

## Kubernetes Library

[[library]]

## Section 1: Resources

- Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

- Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

- Exam Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

Keep the code - 20KLOUD handy while registering for the CKA or CKAD exams at Linux Foundation to get a 20% discount.

- Kubernetes The Hard Way
  https://github.com/mmumshad/kubernetes-the-hard-way

- Certified Kubernetes Administrator (CKA) Course
  https://github.com/kodekloudhub/certified-kubernetes-administrator-course

---

## Section 2: Core Concepts

### Cluster Architecture

[[architecture]]

### Docker vs ContainerD

[[library]]

### Control Plane Components

[[architecture]] > control-plane

### Data Plane Components

[[architecture]] > data-plane

## Section 3: Scheduling

[[architecture]] > control-plane > scheduler

### Manual Scheduling

### Labels and Selectors

### Taints and Tolerations

### Node Selectors

### Node Affinity

### Resource Limits

### DeamonSets

### Static Pods

### Priority Classes

### Multiple Schedulers

### Scheduler Profiles

### Admission Controllers

see: [[controllers]] > admission-controller

#### Custom/Dynamic Admission Controllers

see: [[controllers]] > admission-controller > dynamic_admission_controller

## Section 4: Logging and Monitoring

see: [ckad.md#Section 5: Observability And API Maintenance](../ckad/ckad.md#Section-5-Observability-And-API-Maintenance)

## Section 5: Application Lifecycle Management

### Rolling Updates and Rollbacks

### Configure Applications

### Commands and Arguments

### Environment Variables

### Secrets

### Scaling

#### Manual Scaling

#### Dynamic Scaling - Horizontal

#### Dynamic Scaling - Vertical

## Section 6: Cluster Maintenance

see: [[maintenance]]

### OS Patching

### Kubernetes Releases

### Cluster Upgrade

### Backup & Restore

## Section 7: Security

see: [[security]]

### Basic Authentication (Passwords and Tokens)

### TLS Basics

### TLS in Kubernetes

### View Certificate Details

### Certificates API

### KubeConfig

### API Groups

### Authorization

#### RBAC

### ServiceAccount

### Image Security

### Container Security

### Network Security

## Section 8: Storage

see: [[storage]]

### Docker/Container Storage

### Container Runtime Interface (CRI)

see: [[storage]] > container-storage

### Volumes

### PersistentVolumes

### PersistentVolumeClaims

### StorageClasses

## Section 9: Networking

see: [[networking]]

### Intro to Switching Routing Gateways

### Intro to DNS

### CoreDNS

### Network Namespaces

### Docker Networking

### Container Networking Interface (CNI)

### Cluster Networking

### Pod Networking

### Weave CNI + IPAM

### Service Networking

