# Kubernetes Resources

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Kubernetes Resources](#kubernetes-resources)
  - [Documentation](#documentation)
  - [Labs](#labs)
    - [Kubernetes for Beginners Lab](#kubernetes-for-beginners-lab)
    - [Kubernetes Application Developer - CKAD Lab](#kubernetes-application-developer---ckad-lab)
  - [Tools](#tools)
    - [Kubernetes on the Cloud](#kubernetes-on-the-cloud)
      - [Hosted Solutions](#hosted-solutions)
        - [Google Kubernetes Engine (GKE)](#google-kubernetes-engine-gke)
        - [Amazon Elastic Kubernetes Service (EKS)](#amazon-elastic-kubernetes-service-eks)
        - [Azure Kubernetes Service (AKS)](#azure-kubernetes-service-aks)
    - [Setup K8s cluster](#setup-k8s-cluster)
        - [Kubeadm Setup](#kubeadm-setup)
          - [References](#references)
          - [Kubeadm Steps](#kubeadm-steps)
        - [Minikube Setup](#minikube-setup)
          - [References 1](#references-1)
          - [References 2](#references-2)
          - [Minikube Steps](#minikube-steps)
  - [Kind index](#kind-index)
  - [Commands](#commands)
    - [Select - List - Namespaces](#select---list---namespaces)
      - [Objects](#objects)
      - [Namespaces](#namespaces)
    - [Export](#export)
    - [Creation, Deletion](#creation-deletion)
    - [Replace, Modify, Scale](#replace-modify-scale)
    - [Rollout, Updates](#rollout-updates)

<!-- /code_chunk_output -->

## Documentation

[github.com - Kubernetes](https://github.com/kubernetes/kubernetes)

[kubernetes.io](https://kubernetes.io)

[kubernetes.io - Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

[kubernetes.io docs - Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

[kubernetes.io docs - kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[opensource.com - A guide to Kubernetes architecture](https://opensource.com/article/22/2/kubernetes-architecture)

[opensource.com - A visual guide to Kubernetes networking fundamentals](https://opensource.com/article/22/6/kubernetes-networking-fundamentals?utm_medium=Email&utm_campaign=weekly&sc_cid=7013a00000311fXAAQ)

[opensource.com - A visual map of a Kubernetes deployment](https://opensource.com/article/22/3/visual-map-kubernetes-deployment)

[opensource.com - Migrate databases to Kubernetes using Konveyor](https://opensource.com/article/22/5/migrating-databases-kubernetes-using-konveyor)
[redhat.com - How Kubernetes creates and runs containers: An illustrated guide](https://www.redhat.com/architect/how-kubernetes-creates-runs-containers)

[medium.com - Scaling Kubernetes to Over 4k Nodes and 200k Pods](https://medium.com/paypal-tech/scaling-kubernetes-to-over-4k-nodes-and-200k-pods-29988fad6ed)

[kompose.io - DOCKER COMPOSE TO KUBERNETES](https://kompose.io)

[github.com/dockersamples - docker sample apps](https://github.com/dockersamples)

[Dockerhub - kodekloud voting app images](https://hub.docker.com/r/kodekloud/examplevotingapp_worker)

voting app

- https://github.com/kodekloudhub/example-voting-app

- https://github.com/kodekloudhub/example-voting-app-kubernetes

## Labs

### Kubernetes for Beginners Lab

https://uklabs.kodekloud.com/courses/labs-kubernetes-for-the-absolute-beginners-hands-on

Coupon:

    kk-labs-k8b-lakjg328321095305

### Kubernetes Application Developer - CKAD Lab

https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-application-developer

Coupon:

    udemystudent030485

## Tools

### Kubernetes on the Cloud

- Self Hosted/ Turnkey Solutions

  - Provision, configure, maintain vms
  - Use scripts to deploy clusters
  - AWS example: kops, KubeOne, etc.

- Hosted/ Managed Solutions

  - K8s as a service
  - Provider provisions, maintains vms, installs k8s
  - Cannot access master nodes
  - GCP example: Google Container Engine (GKE)

#### Hosted Solutions

##### Google Kubernetes Engine (GKE)

Pre-reqs:

- Google Cloud Free Tier
  https://cloud.google.com/free/
  https://cloud.google.com/free/docs/gcp-free-tier
  - 12-month free trial: $300 to use on any Google Cloud service
  - Always free: provides limited access to mny common GC resources

Kubernetes on Google Cloud: https://cloud.google.com/kubernetes-engine/docs/

##### Amazon Elastic Kubernetes Service (EKS)

Pre-reqs:

- AWS Free Tier
  https://aws.amazon.com/free

Getting started: https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html

##### Azure Kubernetes Service (AKS)

Pre-reqs:

- Azure account
  https://azure.microsoft.com/en-us/free/free-account-faq

### Setup K8s cluster

deployment methods/modes/options:

- Minikube
- MicroK8s
- Kubeadm

##### Kubeadm Setup

###### References

Oracle VirtualBox: https://www.virtualbox.org/

Vagrant: https://www.vagrantup.com/

Link to download VM images: http://osboxes.org/

Link to kubeadm installation instructions: https://kubernetes.io/docs/setup/independent/install-kubeadm/

The link to Vagrant file:
https://github.com/kodekloudhub/labs-certified-kubernetes-administrator-course

If you are new to VirtualBox or Vagrant, please follow this pre-requisites course to learn about it: https://www.youtube.com/watch?v=Wvf0mBNGjXY

How to use Podman inside of Kubernetes https://www.redhat.com/sysadmin/podman-inside-kubernetes

###### Kubeadm Steps

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
        ## add env var
        ## https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options

- **join workers to master** node
  follow kubeadm bootstrap command output

      kubeadm join ...

##### Minikube Setup

###### References 1

The upcoming demo is completely optional, unless you want to setup a local environment on your laptop. Our hands-on labs will be sufficient for you to practice.

Some links used in the upcoming demo may have changed. Please find the latest links to the documentation topics below:

Install MiniKube: https://kubernetes.io/docs/tasks/tools/install-minikube/

VirtualBox: https://www.virtualbox.org/wiki/Downloads

MiniKube Download page for Windows: https://github.com/kubernetes/minikube/releases

While using Minikube with Virtualization technologies, specify the --vm-driver option like this:

minikube start --vm-driver=<driver_name>
More about it here: https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver

###### References 2

Install and set up the kubectl tool:
https://kubernetes.io/docs/tasks/tools/

Install Minikube:
https://minikube.sigs.k8s.io/docs/start/

Install VirtualBox:
https://www.virtualbox.org/wiki/Downloads
https://www.virtualbox.org/wiki/Linux_Downloads

Minikube Tutorial:
https://kubernetes.io/docs/tutorials/hello-minikube/

If the minikube installation has been done on the macOS, then to access the URL on the local browser, we need to do a few steps to get the service URL to work. Those steps are covered on this documentation page:
https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-service-with-tunnel

###### Minikube Steps

[ServerWorld - Install Minikube to configure Kubernetes Cluster on single node.](https://www.server-world.info/en/note?os=CentOS_Stream_8&p=minikube)

Steps:

- [1] Install a Hypervisor that is supported by Minikube.
  On this example, Install KVM like here of [1] for it.
- [2] Install Snappy, refer to here of [1].
- [3] Install Minikube and other required tools.
- [4] Add users who use Minikube to [libvirt] group.
- [5] Start Minikube with a user who are in [libvirt] group.

## Kind index

## Commands

### Select - List - Namespaces

#### Objects

**list resources**  and their shortcuts. _ex: pods, nodes, namespaces, ..._

    kubectl api-resources

**object selection methods** (not always interchangeable)

    # object selection
    all
    [type] [name]
    [type]/[name]
    -f resource_definition.yaml

**get resource**. _output_format: name, wide, yaml, json_

    kubectl get [resource]
    kubectl get [resource] -o [output_format]

**get resource details**

    kubectl describe [resource]

#### Namespaces

**create namespace**

    kubectl create namespace my-namespace
    kubectl create -f namespace-definition.yaml

**namespace selection**

    kubectl --all-namespaces             [command] [object]
    kubectl --namespace=<namespace-name> [command] [object]

**switch to namespace**

    kubectl config set-context $(kubectl config current-context) --namespace=dev

### Export

**export resource definition file**. _output_format: name, wide, yaml, json_

    kubectl get [resource] -o [output_format] > my-definition.yaml

get **service url**

    minikube service <service> --url

### Creation, Deletion

**run image** on cluster

    kubectl run pod_name --image=image_name

**create resource** from file or stdin

    kubectl create -f file_path


**delete resource**

    kubectl delete [resource]

### Replace, Modify, Scale

**replace a resource**

    kubectl replace [resource]

**apply config to resource**

    kubectl apply -f [config_file]

**edit resource** (opens editor to runtime config)

    kubectl edit [resource]

**scale resource**

for a _deployment_, _replica set_, _replication controller_, or _stateful set_

    kubectl scale --replicas=3 [resource]

**changes application resources**

_changes are to runtime config_

    kubectl set [resource] [deployment] [container_name]=[new_image_name]
      # resources:
        env              Update environment variables on a pod template
        image            Update the image of a pod template
        resources        Update resource requests/limits on objects with pod templates
        selector         Set the selector on a resource
        serviceaccount   Update the service account of a resource
        subject          Update the user, group, or service account in a role binding or cluster role binding

### Rollout, Updates

**controll rollouts**

rollout is valid for _deployments_, _daemonsets_, or _statefulsets_

    kubectl rollout [command] [deployment]
      # commands
        history       View rollout history
        pause         Mark the provided resource as paused
        restart       Restart a resource
        resume        Resume a paused resource
        status        Show the status of the rollout
        undo          Undo a previous rollout
