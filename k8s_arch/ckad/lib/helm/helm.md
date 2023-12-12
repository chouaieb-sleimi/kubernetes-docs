# K8S Helm

tags: #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Helm](#k8s-helm)
  - [Installation](#installation)
  - [Helm Charts](#helm-charts)

<!-- /code_chunk_output -->

---

Automates the creation, packaging, configuration, and deployment of Kubernetes applications by combining definition files into a single reusable package.

## Installation

Installation pre-reqs:

- k8s cluster
- kubectl (configured)

install helm
see: https://helm.sh/docs/intro/install

```bash
# using snap
sudo snap install helm --classic

# using package manager
sudo dnf install helm
sudo apt-get install helm
```

check helm installation

```bash
helm version

# get helm client env information
helm env
```

helm repo commands

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search hub wordpress
helm repo list
```

## Helm Charts

[[chart]]
