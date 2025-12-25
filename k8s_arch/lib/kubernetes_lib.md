# CKAD Library

tags: #k8s

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [CKAD Library](#ckad-library)
  - [Documentation](#documentation)
  - [Labs](#labs)
  - [Images](#images)
  - [Commands](#commands)
  - [Objects List](#objects-list)
  - [Docker vs ContainerD](#docker-vs-containerd)
    - [Containerd CLI Tools:](#containerd-cli-tools)
    - [CRI CLI Tools:](#cri-cli-tools)
  - [Kubernetes on the Cloud](#kubernetes-on-the-cloud)
    - [Hosted Solutions](#hosted-solutions)
      - [Google Kubernetes Engine (GKE)](#google-kubernetes-engine-gke)
      - [Amazon Elastic Kubernetes Service (EKS)](#amazon-elastic-kubernetes-service-eks)
      - [Azure Kubernetes Service (AKS)](#azure-kubernetes-service-aks)
  - [Setup K8s cluster](#setup-k8s-cluster)
    - [Kubeadm Setup](#kubeadm-setup)
      - [Minikube Setup](#minikube-setup)

<!-- /code_chunk_output -->

---

## Documentation

- [kubernetes.io docs - Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)
- [kubernetes.io - basic commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)
- [kubernetes.io docs - kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

- [kompose.io - DOCKER COMPOSE TO KUBERNETES](https://kompose.io)
- [github.com/dockersamples - docker sample apps](https://github.com/dockersamples)
- [Dockerhub - kodekloud voting app images](https://hub.docker.com/r/kodekloud/examplevotingapp_worker)
- [opensource.com - Migrate databases to Kubernetes using Konveyor](https://opensource.com/article/22/5/migrating-databases-kubernetes-using-konveyor)

---

## Labs

- **Kubernetes for Beginners Lab**
  https://uklabs.kodekloud.com/courses/labs-kubernetes-for-the-absolute-beginners-hands-on
  Coupon `kk-labs-k8b-lakjg328321095305`

- **Kubernetes Application Developer - CKAD Lab**
  https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-application-developer
  Coupon `udemystudent030485`

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

## Docker vs ContainerD

K8s **supports containerd** and **not docker**.

Supported K8s runtimes:

- containerd
- CRI-O
- ~~Docker~~ (deprecated)
- Docker Engine (cri-dockerd)

### Containerd CLI Tools:

| Tool | Purpose | Community | Works With |
|------|---------|-----------|-----------|
| **ctr** | Debugging | containerd | `containerd` |
| **nerdctl** | General purpose | containerd | `containerd` |
| **crictl** | Debugging | kubernetes | CRI compatible runtimes |

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

---

## Kubernetes on the Cloud

- Self Hosted/ Turnkey Solutions

  - Provision, configure, maintain vms
  - Use scripts to deploy clusters
  - AWS example: kops, KubeOne, etc.

- Hosted/ Managed Solutions

  - K8s as a service
  - Provider provisions, maintains vms, installs k8s
  - Cannot access master nodes
  - GCP example: Google Container Engine (GKE)

### Hosted Solutions

#### Google Kubernetes Engine (GKE)

Pre-reqs:

- Google Cloud Free Tier
  https://cloud.google.com/free/
  https://cloud.google.com/free/docs/gcp-free-tier
  - 12-month free trial: $300 to use on any Google Cloud service
  - Always free: provides limited access to mny common GC resources

Kubernetes on Google Cloud: https://cloud.google.com/kubernetes-engine/docs/

#### Amazon Elastic Kubernetes Service (EKS)

Pre-reqs:

- AWS Free Tier
  https://aws.amazon.com/free

Getting started: https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html

#### Azure Kubernetes Service (AKS)

Pre-reqs:

- Azure account
  https://azure.microsoft.com/en-us/free/free-account-faq

---

## Setup K8s cluster

deployment methods/modes/options:

- from scratch (manual)
- Minikube
- MicroK8s
- Kubeadm

### Kubeadm Setup

**References:**

- Oracle VirtualBox
  https://www.virtualbox.org/

- Vagrant
  https://www.vagrantup.com/

- Link to download VM images
  http://osboxes.org/

- Link to kubeadm installation instructions
  https://kubernetes.io/docs/setup/independent/install-kubeadm/

- The link to Vagrant file
  https://github.com/kodekloudhub/labs-certified-kubernetes-administrator-course

- If you are new to VirtualBox or Vagrant, please follow this pre-requisites course to learn about it
  https://www.youtube.com/watch?v=Wvf0mBNGjXY

- How to use Podman inside of Kubernetes
  https://www.redhat.com/sysadmin/podman-inside-kubernetes

**Deployment steps:**

- **designate master and worker** nodes
  kodekloud CKA github: https://github.com/kodekloudhub/labs-certified-kubernetes-administrator-course
- install **container runtime**
  https://kubernetes.io/docs/setup/production-environment/container-runtimes/#container-runtimes
- **install kubeadm**: helps bootstraps kubernetes by installing all components on all nodes in the right order
  https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- initialize: **install components on master** server
  kubeadm cmd flags: `--pod-network-cidr` and `--apiserver-advertise-address`
- setup **pod network**
  install addons https://kubernetes.io/docs/concepts/cluster-administration/addons/
  use `Weave Net` addon

      kubectl get ds -A

  - setup IPALLOC_RANGE after installation:
    https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-things-to-watch-out-for

        kubectl get ds -A
        kubectl edit ds weave-net -n kube-system
        # add env var
        # https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options

- **join workers to master** node
  follow kubeadm bootstrap command output

      kubeadm join ...

#### Minikube Setup

**References 1:**

- Install MiniKube
  https://kubernetes.io/docs/tasks/tools/install-minikube/

- VirtualBox
  https://www.virtualbox.org/wiki/Downloads

- MiniKube Download page for Windows
  https://github.com/kubernetes/minikube/releases

- specify the `--vm-driver` option `minikube start --vm-driver=<driver_name>`
  https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver

**References 2:**

- Install and set up the kubectl tool:
  https://kubernetes.io/docs/tasks/tools/

- Install Minikube:
  https://minikube.sigs.k8s.io/docs/start/

- Install VirtualBox:
  https://www.virtualbox.org/wiki/Downloads
  https://www.virtualbox.org/wiki/Linux_Downloads

- Minikube Tutorial:
  https://kubernetes.io/docs/tutorials/hello-minikube/

- If the minikube installation has been done on the macOS, then to access the URL on the local browser, we need to do a few steps to get the service URL to work. Those steps are covered on this documentation page:
  https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-service-with-tunnel

**Deployment steps:**

[server-world.info - Install Minikube to configure Kubernetes Cluster on single node.](https://www.server-world.info/en/note?os=CentOS_Stream_8&p=minikube)

Steps:

1. Install a Hypervisor that is supported by Minikube.
   On this example, Install KVM like here of [1] for it.
2. Install Snappy, refer to here of [1].
3. Install Minikube and other required tools.
4. Add users who use Minikube to `libvirt` group.
5. Start Minikube with a user who are in `libvirt` group.
