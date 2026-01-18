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
  - [Scheduling](#scheduling)
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
  - [Logging and Monitoring](#logging-and-monitoring)
  - [Application Lifecycle Management](#application-lifecycle-management)
    - [Rolling Updates and Rollbacks](#rolling-updates-and-rollbacks)
    - [Configure Applications](#configure-applications)
    - [Commands and Arguments](#commands-and-arguments)
    - [Environment Variables](#environment-variables)
    - [Secrets](#secrets)
    - [Scaling](#scaling)
      - [Manual Scaling](#manual-scaling)
      - [Dynamic Scaling - Horizontal](#dynamic-scaling---horizontal)
      - [Dynamic Scaling - Vertical](#dynamic-scaling---vertical)
  - [Cluster Maintenance](#cluster-maintenance)
    - [OS Patching](#os-patching)
    - [Kubernetes Releases](#kubernetes-releases)
    - [Cluster Upgrade](#cluster-upgrade)
    - [Backup & Restore](#backup--restore)
  - [Security](#security)
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

[[control-plane]]

### Data Plane Components

[[data-plane]]

## Scheduling

see: [[scheduler]]

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

see: [[admission-controller.md]]

#### Custom/Dynamic Admission Controllers

see: [dynamic_admission_controller.md](../architecture/control-plane/controllers/dynamic_admission_controller.md)

## Logging and Monitoring

see: [ckad.md#Section 5: Observability And API Maintenance](../ckad/ckad.md#Section-5-Observability-And-API-Maintenance)

## Application Lifecycle Management

### Rolling Updates and Rollbacks

### Configure Applications

### Commands and Arguments

### Environment Variables

### Secrets

### Scaling

#### Manual Scaling

#### Dynamic Scaling - Horizontal

#### Dynamic Scaling - Vertical

## Cluster Maintenance

see: [[maintenance]]

### OS Patching

### Kubernetes Releases

### Cluster Upgrade

### Backup & Restore

## Security

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

