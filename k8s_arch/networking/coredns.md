# K8S Networking

tags: #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [CoreDNS Components](#coredns-components)
- [Setup of CoreDNS](#setup-of-coredns)
- [CoreDNS in Kubernetes](#coredns-in-kubernetes)

<!-- /code_chunk_output -->

---

## CoreDNS Components

- **deployment**: `coreDNS`
  - **pods**: namespace=`kube-system`
- **configmap**: holds `Corefile` definition
- **service**: `kube-dns`, port 53
  service IP is pointed to by each pod's `resolv.conf`
- **serviceaccount**: `coreDNS`
- **clusterRole/clusterRoleBinding**: `core-dns/kube-dns`

## Setup of CoreDNS

**download and run**

- by default, listens on port 53

  ```bash
  wget https://github.com/coredns/coredns/releases/download/v1.7.0/coredns_1.7.0_linux_amd64.tgz
  coredns_1.7.0_linux_amd64.tgz
  tar -xzvf coredns_1.7.0_linux_amd64.tgz
  coredns
  ./coredns
  ```

- **manual configuration**
  - add entries into the `/etc/hosts` file
  - Configure `/etc/coredns/Corefile` to use hosts entries
  - start/restart/reload CoreDNS

  ```properties
  .:53: {
    cache 30
    log
    errors

    # use /etc/hosts
    hosts   /etc/hosts {
      reload 1m
      fallthrough
    }

    # forward unresolved queries to host's resolver
    forward . /etc/erolv.conf {
      max_concurrent 1000
    }
  }
  ```

## CoreDNS in Kubernetes

- is run as a deployment w/ `replicas: 2` for redundency
- kubelet configures pods' `/etc/resolv.conf` with:
  - **CoreDNS service** cluster-ip
  - **CoreDNS search domains** (`cluster.local` `svc.cluster.local` `default.svc.cluster.local`)
    > CoreDNS's cluster-ip is configured in the kubelet (`--cluster-dns` + `--cluster-domain`)
- `/etc/coredns/Corefile` configmap data:

```properties
.:53: {
  log
  errors

  kubernetes cluster.local in-addr.arpa ip6.arpa {
    # creates pod records (iip-dash-format)
    pods insecure
    upstream
    fallthrough in-addr.arpa ip6.arpa
    ttl 30
  }

  prometheus :9153

  # forward unresolved queries to node's resolver
  proxy . /etc/resolv.conf

  cache 30
  reload
}
```
