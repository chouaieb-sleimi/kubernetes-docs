# K8S Documentation

tags: #docs

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Documentation](#k8s-documentation)
  - [References](#references)
  - [Tutorials](#tutorials)
  - [Architecture](#architecture)
  - [Tools and Utils](#tools-and-utils)
    - [Images](#images)

<!-- /code_chunk_output -->

---

## References

- [kubernetes.io](https://kubernetes.io)
- [github.com - Kubernetes](https://github.com/kubernetes/kubernetes)
- [helm docs](https://helm.sh/docs/)

## Tutorials

## Architecture

- [kubernetes.io - Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [opensource.com - A guide to Kubernetes architecture](https://opensource.com/article/22/2/kubernetes-architecture)
- [opensource.com - A visual guide to Kubernetes networking fundamentals](https://opensource.com/article/22/6/kubernetes-networking-fundamentals?utm_medium=Email&utm_campaign=weekly&sc_cid=7013a00000311fXAAQ)
- [opensource.com - A visual map of a Kubernetes deployment](https://opensource.com/article/22/3/visual-map-kubernetes-deployment)
- [redhat.com - How Kubernetes creates and runs containers: An illustrated guide](https://www.redhat.com/architect/how-kubernetes-creates-runs-containers)
- [medium.com - Scaling Kubernetes to Over 4k Nodes and 200k Pods](https://medium.com/paypal-tech/scaling-kubernetes-to-over-4k-nodes-and-200k-pods-29988fad6ed)

## Tools and Utils

- [kubernetes.io docs - Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)
- [kubernetes.io - basic commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)
- [kubernetes.io docs - kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

- [kompose.io - DOCKER COMPOSE TO KUBERNETES](https://kompose.io)
- [github.com/dockersamples - docker sample apps](https://github.com/dockersamples)
- [Dockerhub - kodekloud voting app images](https://hub.docker.com/r/kodekloud/examplevotingapp_worker)
- [opensource.com - Migrate databases to Kubernetes using Konveyor](https://opensource.com/article/22/5/migrating-databases-kubernetes-using-konveyor)

### Images

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
