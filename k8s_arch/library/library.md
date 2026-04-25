# Kubernetes Library

tags: #k8s

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Documentation](#documentation)
- [Certs/ Labs](#certs-labs)
- [Images](#images)
- [Commands](#commands)
- [Objects List](#objects-list)
- [Tools & Utils](#tools--utils)
  - [Dev & CLI](#dev--cli)
  - [Helm](#helm)
  - [Kustomize](#kustomize)
- [Docker vs ContainerD](#docker-vs-containerd)
  - [Containerd CLI Tools:](#containerd-cli-tools)
  - [CRI CLI Tools:](#cri-cli-tools)

<!-- /code_chunk_output -->

---

## Documentation

- [kubernetes.io docs - Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)
- [kubernetes.io - basic commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)
- [kubernetes.io docs - kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [kubernetes.io docs - declarative config](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
-
- [kompose.io - DOCKER COMPOSE TO KUBERNETES](https://kompose.io)
- [github.com/dockersamples - docker sample apps](https://github.com/dockersamples)
- [Dockerhub - kodekloud voting app images](https://hub.docker.com/r/kodekloud/examplevotingapp_worker)
- [opensource.com - Migrate databases to Kubernetes using Konveyor](https://opensource.com/article/22/5/migrating-databases-kubernetes-using-konveyor)
-
- [CNCF Exam Corriculum](https://github.com/cncf/curriculum)
- [Linux Foundation Exam Tips](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad)
- [Linux Foundation Handbook](https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2/exam-preparation-checklist)

---

## Certs/ Labs

- **Kubernetes for Beginners Lab**
  https://uklabs.kodekloud.com/courses/labs-kubernetes-for-the-absolute-beginners-hands-on
  Coupon `kk-labs-k8b-lakjg328321095305`

- **Kubernetes Application Developer - CKAD Lab**
  https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-application-developer
  Coupon `udemystudent030485`
- **Kubernetes Administration - CKA Lab**
  https://learn.kodekloud.com/user/courses/udemy-labs-certified-kubernetes-administrator-with-practice-tests
  Coupon `kk-labs-cka-lakjg328321095305`

- **Kubernetes Networking Lab**
  https://uklabs.kodekloud.com/courses/labs-kubernetes-networking
  Coupon `kk-labs-k8n-lakjg328321095305`

- **Kubernetes Challenges**
  https://kodekloud.com/courses/kubernetes-challenge

- **DevopsCube Ckad Exam Study Guide**
  https://devopscube.com/ckad-exam-study-guide/

- **KLLR SHLL - Linux Foundation Exam Simulators**
  https://killer.sh

- **KLLR CODA - Interactive environments**
  https://killercoda.com

---

## Images

**utilities images**

- google cluster end-to-end tests: `gcr.io/kubernetes-e2e-test-images/dnsutils`

**sample admission controller servers**

- sample k8s GOLang controller: `https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go`

**Kodekloud images:**

- ServiceAccount demo image: `gcr.io/kodekloud/customimage/my-kubernetes-dashboard`
- Secrets demo image: `kodekloud/simple-webapp-mysql`
- Readiness and Liveness Probes demo image: `kodekloud/webapp-delayed-start`
- Container Logging demo image: `kodekloud/event-simulator`
- Jobs and CronJobs demo image: `kodekloud/throw-dice`
- Ingress demo images:
  - `kodekloud/ecommerce:apparels`
  - `kodekloud/ecommerce:video`
  - `kodekloud/ecommerce:food`
  - `kodekloud/ecommerce:404`
- log events demo image: `kodekloud/event-simulator`
- custom admission controllers demo image: `stackrox/admission-controller-webhook-demo:latest`
- voting app
  - https://hub.docker.com/r/kodekloud/examplevotingapp_worker
  - https://github.com/kodekloudhub/example-voting-app
  - https://github.com/kodekloudhub/example-voting-app-kubernetes

---

## Commands

[[commands.md]]

---

## Objects List

[[api-resources-ckad]]

---

## Tools & Utils

### Dev & CLI

- kui (desktop-deprecated)
- k9s (desktop)
- kunobi (desktop)
- Headlamp (desktop)
- Lens (IDE)
- OpenLens (opensource Lens)


### Helm

[[helm]]


### Kustomize

[[kustomize]]

---

## Docker vs ContainerD

K8s **supports containerd** and **not docker**.

Supported K8s runtimes:

- containerd
- CRI-O
- ~~Docker~~ (deprecated)
- Docker Engine (cri-dockerd)

### Containerd CLI Tools:

| Tool        | Purpose         | Community  | Works With              |
| ----------- | --------------- | ---------- | ----------------------- |
| **ctr**     | Debugging       | containerd | `containerd`            |
| **nerdctl** | General purpose | containerd | `containerd`            |
| **crictl**  | Debugging       | kubernetes | CRI compatible runtimes |

- **ctr**
  - comes w/ containerd
  - not user friendly
  - limited features

- **nerdctl**
  - docker-like cli
  - supports docker-compose
  - supports containerd features:
    - encrypted container images
    - lazy pulling
    - image signing and verifying
    - namespaces w/ k8s

### CRI CLI Tools:

- **crictl**
  - installed separately
  - inspect and debug runtimes
    - not to create containers
      (any created containers will be removed by kubelet)
  - cross containre runtimes
  - default sockets:

    ```bash
    # crictl --runtime-endpoint
    unix:///run/containerd/containerd.sock
    unix:///run/crio/crio.sock
    unix:///var/run/cri-dockerd.sock

    # set endpoint w/ env var
    export CRICTL_RUNTIME_ENDPOINT=unix:///run/containerd/containerd.sock
    ```
