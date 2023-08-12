# Certified Kubernetes Application Developer - CKAD

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Certified Kubernetes Application Developer - CKAD](#certified-kubernetes-application-developer---ckad)
- [Section 1: Overview](#section-1-overview)
  - [Resources](#resources)
  - [Section 2: Core Concepts](#section-2-core-concepts)
    - [Docker vs ContainerD](#docker-vs-containerd)
    - [Containerd CLIs:](#containerd-clis)
    - [Namespaces](#namespaces)

<!-- /code_chunk_output -->

# Section 1: Overview

## Resources

- Certified Kubernetes Application Developer: https://www.cncf.io/certification/ckad/

- Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

- Exam Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

Keep the code - 20KLOUD handy while registering for the CKA or CKAD exams at Linux Foundation to get a 20% discount.

- Kubernetes The Hard Way
  https://github.com/mmumshad/kubernetes-the-hard-way

## Section 2: Core Concepts

### Docker vs ContainerD

![docker vs. podman](img/section-02_docker-vs-podman.jpeg)
K8s **supports containerd** and **not docker**.

Supported K8s runtimes:

- containerd
- CRI-O
- Docker Engine (cri-dockerd)

### Containerd CLIs:

- **ctr**
  **purpose: debugging**
  **community: containerd**
  **works with: containerd**

  - comes w/ containerd
  - not user friendly
  - limited features

- **nerdctl**
  **purpose: general purrpose**
  **community: containerd**
  **works with: containerd**
  - docker-like cli
  - supports docker-compose
  - supports containerd features:
    - encrypted container images
    - lazy pulling
    - image signing and verifying
    - namespaces w/ k8s

**CRI CLIs:**

- **crictl**
  **purpose: debugging**
  **community: kubernetes**
  **works with: CRI compatible runtimes**
  - installed separately
  - inspect and debug runtimes
    - not to create containers
      (any created containers will be removed by kubelet)
  - cross containre runtimes
  - should manually set runtime endpoints (`crictl --runtime-endpoint`):
    unix:///run/containerd/containerd.sock
    unix:///run/crio/crio.sock
    unix:///var/run/cri-dockerd.sock

### Namespaces

Provides a mechanism for isolating groups of resources within a single cluster.

- They need to be unique within a namespace, but not across namespaces.
- Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

**Cross-namespace object name format:**
format: `<object-name>.<namespace-name>.<object-type>.<cluster-domain>`
example: `db-service.dev.service.cluster.local`
