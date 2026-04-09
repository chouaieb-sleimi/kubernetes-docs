# K8S Architecture

tags: #arch

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [References](#references)
- [Documentation](#documentation)
- [K8S Components](#k8s-components)
  - [Control Plane](#control-plane)
  - [Nodes](#nodes)
  - [Namespace](#namespace)
  - [Operator Framework](#operator-framework)
- [Design A Kubernetes Cluster](#design-a-kubernetes-cluster)
- [Kubernetes on the Cloud](#kubernetes-on-the-cloud)
  - [Hosted Solutions](#hosted-solutions)
    - [Google Kubernetes Engine (GKE)](#google-kubernetes-engine-gke)
    - [Amazon Elastic Kubernetes Service (EKS)](#amazon-elastic-kubernetes-service-eks)
    - [Azure Kubernetes Service (AKS)](#azure-kubernetes-service-aks)
- [High Availability](#high-availability)
  - [ETCD in HA](#etcd-in-ha)
- [Deploy Kubernetes Cluster](#deploy-kubernetes-cluster)
  - [Minikube Setup](#minikube-setup)
  - [Hard Way](#hard-way)
  - [Kubeadm Way](#kubeadm-way)
    - [Deployment overview](#deployment-overview)
    - [Deployment process](#deployment-process)

<!-- /code_chunk_output -->

---

## References

- [kubernetes.io](https://kubernetes.io)
- [github.com - Kubernetes](https://github.com/kubernetes/kubernetes)
- [helm docs](https://helm.sh/docs/)

## Documentation

- [kubernetes.io - Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [opensource.com - A guide to Kubernetes architecture](https://opensource.com/article/22/2/kubernetes-architecture)
- [opensource.com - A visual guide to Kubernetes networking fundamentals](https://opensource.com/article/22/6/kubernetes-networking-fundamentals?utm_medium=Email&utm_campaign=weekly&sc_cid=7013a00000311fXAAQ)
- [opensource.com - A visual map of a Kubernetes deployment](https://opensource.com/article/22/3/visual-map-kubernetes-deployment)
- [redhat.com - How Kubernetes creates and runs containers: An illustrated guide](https://www.redhat.com/architect/how-kubernetes-creates-runs-containers)
- [medium.com - Scaling Kubernetes to Over 4k Nodes and 200k Pods](https://medium.com/paypal-tech/scaling-kubernetes-to-over-4k-nodes-and-200k-pods-29988fad6ed)

---

## K8S Components

### Control Plane

see [[control-plane]]

**components:**

- api-server
- etcd
- scheduler
- controller manager

### Nodes

see [[data-plane]]

**components:**

- kubelet
- kube-proxy
- container runtime

### Namespace

see [namespace.md](namespace.md)

**components:**

- resource quota
- network policy
- limit range

### Operator Framework

[[operator]]

---

## Design A Kubernetes Cluster

**kubernetes solutions**

- **Turnkey solutions**
  examples:
  - k8s on AWS using KOPS
  - openshift
  - Cloud Foundry - Container Runtime
  - VMWare Cloud PKS
  - Vagrant (cloud deployment scripts)

  you:
  - provision/configure VMs
  - deploy cluster
  - maintain VMs

- **Hosted solutions**
  examples:
  - Google GCE (GKE)
  - Openshift Online
  - Azure Kubernetes Service
  - Amazon ECS (EKS)

  k8s-as-a-service:
  - managed VM provision
  - managed cluster deployment
  - managed VM maintenance

**cluster limitations**

- max nodes per cluster: 5000
- max pods per cluster: 150,000
- max totalcontainers: 300,000
- max pods per node: 100

**storage recommendations**

- use SSD-backed storage
- use network-attached storage (NAS) for high availability/ concurrent access
- use SC/PV/PVC
- label nodes w/ specific disk types
- use node selectors to assing pods to specific node/storage

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

## High Availability

**multiple master components**

- **api-server** (active-active)
- **controller-manager** (active-passive)
  uses leader-elect mechanism
- **scheduler** (active-passive)
  uses leader-elect mechanism
- **etcd** (2 topologies)
  - **stacked topology:** etcd is deployed in control-plane nodes
    easier to setup+manage, fewer nodes, risk during failures
  - **external topology:** etcd is deployed in separate nodes
    harder to setup+manage, more servers, less risk during failures

**leader-elect mechanism**

- when component start: leader gets lock/lease on an endpoint object
  - `--leader-elect` defaults to `true`
  - first component that updates the endpoint: gains lease
    -> become active
- holds lease object for the `--leader-elect-lease-duration` period
- renews lease every `--leader-elect-renew-deadline` period
- every active/passive try to become leader every `--leader-elect-retry-period` period

```bash
kube-controller-manager --leader-elect true \
                        --leader-elect-lease-duration 15s \
                        --leader-elect-renew-deadline 10s \
                        --leader-elect-retry-period 2s \
                        [other options]

kube-scheduler --leader-elect true \
                        --leader-elect-lease-duration 15s \
                        --leader-elect-renew-deadline 10s \
                        --leader-elect-retry-period 2s \
                        [other options]
```

### ETCD in HA

see: [[control-plane]] > etcd

---

## Deploy Kubernetes Cluster

deployment methods/modes/options:

- from scratch (manual)
- Minikube
- MicroK8s
- Kubeadm

### Minikube Setup

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

### Hard Way

see: KodeKloud Youtube - Install kubernetes from scratch
https://www.youtube.com/playlist?list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo
see: KodeKloud GitHub - kubernetes the hard way
https://github.com/mmumshad/kubernetes-the-hard-way.git

### Kubeadm Way

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

#### Deployment overview

1.  designate **master and worker** nodes
    kodekloud CKA github: https://github.com/kodekloudhub/labs-certified-kubernetes-administrator-course
1.  install component: **container-runtime**
    https://kubernetes.io/docs/setup/production-environment/container-runtimes/#container-runtimes
1.  install tool: **kubeadm**: helps bootstraps kubernetes by installing all components on all nodes in the right order
    https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
1.  initialize master server: **install components on master**
    - install **kubeapi-server**, **scheduler**, **controller-manager**, **etcd**
      kubeadm cmd flags: `--pod-network-cidr` and `--apiserver-advertise-address`
1.  setup **pod network**
    use `Weave Net` addon
    install addons https://kubernetes.io/docs/concepts/cluster-administration/addons/
    `kubectl get ds -A`
    - setup IPALLOC_RANGE after installation:
      https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-things-to-watch-out-for

      ```bash
      kubectl get ds -A
      kubectl edit ds weave-net -n kube-system
      # add env var
      # https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options
      ```

#### Deployment process

see: Bootstrapping clusters with kubeadm -
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/

1. Install required software
   - VirtualBox: https://www.virtualbox.org/
   - Vagrant: https://developer.hashicorp.com/vagrant/downloads

1. Clone this repository

   ```
   git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
   cd certified-kubernetes-administrator-course
   ```

1. Bring up the virtual machines

   ```
   vagrant up
   ```

   This will start 3 virtual machines named
   - `kubemaster` - where we will install the control plane
   - `kubenode01`
   - `kubenode02`

1. Check you can SSH to each VM

   Note: To exit from VM's ssh session, enter `exit`

   ```
   vagrant ssh kubemaster
   ```

   ```
   vagrant ssh kubenode01
   ```

   ```
   vagrant ssh kubenode02
   ```

1. Initialize the nodes.

   The following steps must be performed on each of the three nodes, so `ssh` to `kubemaster` and run the steps, then to `kubenode01`, then to `kubenode02`
   1. Configure kernel parameters

      ```
      {
      cat <<EOF | sudo tee /etc/modules-load.d/11-k8s.conf
      br_netfilter
      EOF

      sudo modprobe br_netfilter

      cat <<EOF | sudo tee /etc/sysctl.d/11-k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      EOF

      sudo sysctl --system
      }
      ```

   1. Install `containerd` container driver and associated tooling

      ```bash
      {
          sudo apt update
          sudo apt install -y apt-transport-https ca-certificates curl
          sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
          echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
          sudo apt-get install -y containerd
          #sudo mkdir -p /opt/cni/bin
          #wget -q --https-only \
          #  https://github.com/containernetworking/plugins/releases/download/v0.8.#6/cni-plugins-linux-amd64-v0.8.6.tgz \
          #  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.25.0/#crictl-v1.25.0-linux-amd64.tar.gz
          #sudo tar -xzf cni-plugins-linux-amd64-v0.8.6.tgz -C /opt/cni/bin
          #sudo tar -xzf crictl-v1.25.0-linux-amd64.tar.gz -C /usr/local/bin
      }
      ```

   1. Install Kubernetes software

      This will install the latest version

      ```bash
      {
      sudo  curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

      echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.

      sudo apt update

      sudo apt-get install -y kubelet kubeadm kubectl
      sudo apt-mark hold kubelet kubeadm kubectl

      # Configure crictl so it doesn't print ugly warning messages
      sudo crictl config \
          --set runtime-endpoint=unix:///run/containerd/containerd.sock \
          --set image-endpoint=unix:///run/containerd/containerd.sock
      }
      ```

1. Initialize controlplane node
   1. Get the IP address of the `eth0` adapter of the controlplane

      ```
      ip addr show dev enp0s8
      ```

      Take the value printed for `inet` in the output. This should be:

      > 192.168.56.11

   1. Create a config file for `kubeadm` to get settings from

      ```yaml
      kind: ClusterConfiguration
      apiVersion: kubeadm.k8s.io/v1beta3
      kubernetesVersion: v1.25.4 # <- At time of writing. Change as appropriate
      controlPlaneEndpoint: 192.168.56.11:6443
      networking:
        serviceSubnet: "10.96.0.0/16"
        podSubnet: "10.244.0.0/16"
        dnsDomain: "cluster.local"
      controllerManager:
        extraArgs:
          "node-cidr-mask-size": "24"
      apiServer:
        extraArgs:
          authorization-mode: "Node,RBAC"
        certSANs:
          - "192.168.56.11"
          - "kubemaster"
          - "kubernetes"

      ---
      kind: KubeletConfiguration
      apiVersion: kubelet.config.k8s.io/v1beta1
      cgroupDriver: systemd
      serverTLSBootstrap: true
      ```

   1. Run `kubeadm init` using the IP address determined above for `--apiserver-advertise-address`

      ```
      sudo kubeadm init \
         --apiserver-cert-extra-sans=kubemaster01 \
         --apiserver-advertise-address 192.168.56.11 \
         --pod-network-cidr=10.244.0.0/16
      ```

      Note the `kubeadm join` command output at the end of this run. You will require it for the step `Initialize the worker nodes` below

   1. Set up the default kubeconfig file

      ```
      {
      mkdir ~/.kube
      sudo cp /etc/kubernetes/admin.conf ~/.kube/config
      sudo chown vagrant:vagrant ~/.kube/config
      }
      ```

1. Initialize the worker nodes

   The following steps must be performed on both worker nodes, so `ssh` to `kubenode01` and run the steps, then to `kubenode02`
   - Paste the `kubeadm join` command from above step to the command prompt and enter it.
