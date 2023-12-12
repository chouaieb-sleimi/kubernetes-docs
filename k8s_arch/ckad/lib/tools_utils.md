# K8S Tools

tags: #tools_utils

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Tools](#k8s-tools)
  - [Commands](#commands)
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

## Commands

[[cmd]]

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

## Setup K8s cluster

deployment methods/modes/options:

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
