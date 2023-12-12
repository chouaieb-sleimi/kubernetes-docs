# K8S Security

tags: #objects #security

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Security](#k8s-security)
  - [Container Security](#container-security)
  - [SecurityContext](#securitycontext)
  - [Authentication](#authentication)
  - [Authorization](#authorization)
  - [NetworkPolicy](#networkpolicy)
  - [Operator Framework](#operator-framework)
    - [CustomControllers](#customcontrollers)
    - [CustomResourceDefinition](#customresourcedefinition)

<!-- /code_chunk_output -->

---

Secure:

- **hosts**

  - ssh root access disabled
  - ssh password access disabled
  - ssh key based auth

- **cluster:**

  - Set up certs for inter-cluster-components TLS communication
  - Secure inter-pod comm using NetworkPolicies

**Authentication:** (who can access)

- Files - Username and password
- Files - Username and tokens
- Certificates
- Axternal auth providers - LDAP
- Service accounts (machines)

**Authorization:** (what can they do)

- RBAC (Role Based Access Control)
- ABAC (Atribute Based Access Control)
- Node Auth
- Webhook mode

## Container Security

list user capabilities

    /usr/include/linux/capability.h

override user privileges in podman run cmd

    # add privilege flag
    podman run --cap-add MAC_ADMIN ubuntu

    # drop privilege flag
    podman run --cap-drop KILL ubuntu

    # add ALL privileges
    podman run --privileged ubuntu

## SecurityContext

[[security_context]]

## Authentication

[[user-auth]]

## Authorization

[[user-auth]]

## NetworkPolicy

[[networkPolicy]]

## Operator Framework

[[operator]]

### CustomControllers

### CustomResourceDefinition
