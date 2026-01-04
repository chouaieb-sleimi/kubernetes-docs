# K8S Security

tags: #security

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Security](#k8s-security)
  - [Concepts and Components](#concepts-and-components)
    - [Authentication](#authentication)
      - [ServiceAccount](#serviceaccount)
    - [Authorization](#authorization)
      - [RBAC](#rbac)
    - [AdmissionController](#admissioncontroller)
      - [Dynamic AdmissionController](#dynamic-admissioncontroller)
    - [Request Flow](#request-flow)
    - [TLS Basics](#tls-basics)
  - [TLS in Kubernetes](#tls-in-kubernetes)
  - [Container Security](#container-security)
    - [User Capabilities](#user-capabilities)
    - [SecurityContext](#securitycontext)
  - [Network Security](#network-security)
    - [NetworkPolicy](#networkpolicy)
  - [Operator Framework](#operator-framework)
    - [CustomControllers](#customcontrollers)
    - [CustomResourceDefinition](#customresourcedefinition)

<!-- /code_chunk_output -->

---

## Concepts and Components

**Security primitives (What to secure)**

- hosts
  - ssh root access disabled
  - ssh password access disabled
  - ssh key based auth
- cluster:
  - authentication
    - setup certs for inter-components TLS communication
  - authorization
  - secure inter-pod communication (NetworkPolicies)

### Authentication

who can access
[[authentication]]

- Files - Username and password (deprecated in 1.19)
- Files - Username and tokens (deprecated in 1.19)
- Certificates
- External auth providers - LDAP
- Service accounts (machines)

#### ServiceAccount

[[authentication]] > serviceaccount

### Authorization

what can they do
[[authorization]]

- RBAC (Role Based Access Control)
- ABAC (Atribute Based Access Control)
- Node Authorization
- Webhook mode

#### RBAC


### AdmissionController

[[admission_controller]]

#### Dynamic AdmissionController

[[admission_controller]] > dynamic_admission_controller

### Request Flow

1. **authentication**
   - certs (better for service accounts)
   - tokens (better for users)
     - static token file
     - service account token
   - oidc
2. **authorization** (rbac, abac, webhook)
   - **rbac:** clusterroles, clusterrolebindings, roles, rolebindings
   - **abac:** json policy file
   - **webhook:** external service
3. **admission controllers**
   - built-in controllers (ex: namespace lifecycle, resource quota, pod security policy)
   - **types:**
     - **mutating admission controllers:** run first, can modify requests
     - **validating admission controllers:** run after mutating, cannot modify requests

### TLS Basics

[[tls-basics]]

## TLS in Kubernetes

[[tls-kubernetes]]

---

## Container Security

### User Capabilities

list user capabilities

    /usr/include/linux/capability.h

override user privileges in podman run cmd

    # add privilege flag
    podman run --cap-add MAC_ADMIN ubuntu

    # drop privilege flag
    podman run --cap-drop KILL ubuntu

    # add ALL privileges
    podman run --privileged ubuntu

### SecurityContext

[[security_context]]

---

## Network Security

### NetworkPolicy

[[networkPolicy]]

---

## Operator Framework

[[operator]]

### CustomControllers

### CustomResourceDefinition
