# Kubernetes Manual Installation

tags: #k8s

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Control Plane Components](#control-plane-components)
  - [API-Server](#api-server)
  - [ETCD](#etcd)
  - [Controller-Manager](#controller-manager)
  - [Scheduler](#scheduler)
- [Data Plane Components](#data-plane-components)
  - [Kubelet](#kubelet)
- [Kube-Proxy](#kube-proxy)

<!-- /code_chunk_output -->

---

## Control Plane Components

### API-Server

1. download

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kube-apiserver
```

2. systemd service file `kube-apiserver.service`

```bash
[Unit]
Description=Kubernetes API Server
Documentation=https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
After=network.target
Wants=network.target
[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=<MASTER_IP> \
  --allow-privileged=true \
  --apiserver-count=1 \
  --authorization-mode=Node,RBAC \
  --enable-admission-plugins=NodeRestriction \
  --etcd-servers=http://<ETCD_IP>:2379 \
  --service-cluster-ip-range=<SERVICE_CLUSTER_IP_RANGE> \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
  --tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
  --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt \
  --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key \
  --client-ca-file=/etc/kubernetes/pki/ca.crt \
  --service-account-key-file=/etc/kubernetes/pki/sa.pub \
  --v=28
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```

### ETCD

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

5. systemd service file `etcd.service`

   ```bash
    [Unit]
    Description=etcd key-value store
    Documentation=https://github.com/etcd-io/etcd
    After=network.target
    Wants=network.target
    [Service]
    ExecStart=/usr/local/bin/etcd \
      --name <ETCD_NAME> \
      --data-dir /var/lib/etcd \
      --listen-client-urls http://<ETCD_IP>:2379 \
      --advertise-client-urls http://<ETCD_IP>:2379
    Restart=always
    RestartSec=10s
    [Install]
    WantedBy=multi-user.target
   ```

### Controller-Manager

1. download

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kube-controller-manager
```

2. systemd service file `kube-controller-manager.service`

```bash
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/
After=network.target
Wants=network.target
[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --bind-address=<MASTER_IP> \
   --cluster-cidr=<POD_CIDR> \
   --cluster-name=kubernetes \
   --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \
   --cluster-signing-key-file=/etc/kubernetes/pki/ca.key \
   --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
   --leader-elect=true \
   --root-ca-file=/etc/kubernetes/pki/ca.crt \
   --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
   --service-cluster-ip-range=<SERVICE_CLUSTER_IP_RANGE> \
   --use-service-account-credentials=true \
   --v=28 \
   --controllers=*,bootstrapsigner,tokencleaner,-cronjob \
   --node-monitor-period=5s \
   --node-monitor-grace-period=15s \
   --pod-eviction-timeout=30s
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```

### Scheduler

1. download

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kube-scheduler
```

2. systemd service file `kube-scheduler.service`

```bash
[Unit]
Description=Kubernetes Scheduler
Documentation=https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/
After=network.target
Wants=network.target
[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --address=<MASTER_IP> \
  --kubeconfig=/etc/kubernetes/scheduler.kubeconfig \
  --leader-elect=true \
  --v=28
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```

```bash
# /etc/kubernetes/scheduler.yaml

spec:
  containers:
  - command:
    - kube-scheduler
    - --address=<MASTER_IP>
    - --leader-elect=true
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.kubeconfig
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.kubeconfig
    - --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
    image: k8s.gcr.io/kube-scheduler:v1.20.0
    name: kube-scheduler
```

## Data Plane Components

### Kubelet

- **always installed manually** (even with kubeadm)

1. download

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kubelet
```

2. systemd service file `kubelet.service`

```bash
[Unit]
Description=Kubernetes Kubelet
Documentation=https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
After=network.target
Wants=network.target
[Service]
ExecStart=/usr/local/bin/kubelet \
  --config /var/lib/kubelet/kubelet-config.yaml \
  --cluster-dns 10.10.10.10 \
  --cluster-domain cluster.local \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
  --register-node=true \
  --network-plugin=cni \
  --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf \
  --cgroup-driver=systemd \
  --pod-infra-container-image=k8s.gcr.io/pause:3.2 \
  --v=28
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```

## Kube-Proxy

1. download

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kube-proxy
```

2. systemd service file `kube-proxy.service`

```bash
[Unit]
Description=Kubernetes Kube-Proxy
Documentation=https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/
After=network.target
Wants=network.target
[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig \
  --v=28
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```
