# Kubernetes - ETCD

tags: #arch #controlplane #etcd

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Kubernetes - ETCD](#kubernetes---etcd)
  - [Installation](#installation)

<!-- /code_chunk_output -->

---

## Installation

1. download etcd binary from the official site:

   ```bash
   wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-amd64.tar.gz
    tar xvf etcd-v3.5.0-linux-amd64.tar.gz
    cd etcd-v3.5.0-linux-amd64
   ```

2. move the etcd and etcdctl binaries to /usr/local/bin (optional):

   ```bash
   sudo mv etcd etcdctl /usr/local/bin/
   ```

3. verify the installation:

   ```bash
   etcd --version
   etcdctl version
   ```

4. run etcd server (for testing purposes):

   ```bash
   etcd
   ```