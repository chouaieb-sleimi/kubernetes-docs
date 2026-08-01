# K8S Commands

tags: #tools_utils

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [System Architecture](#system-architecture)
  - [Common Linux paths for Kubernetes nodes](#common-linux-paths-for-kubernetes-nodes)
    - [Controller node (kubeadm layout)](#controller-node-kubeadm-layout)
    - [Worker node (kubeadm layout)](#worker-node-kubeadm-layout)
- [Docker](#docker)
- [Cluster Maintenance](#cluster-maintenance)
  - [Component and Node Management](#component-and-node-management)
  - [Cluster Upgrade](#cluster-upgrade)
  - [ETCD](#etcd)
  - [Backup & Restore](#backup--restore)
  - [TLS Setup](#tls-setup)
  - [TLS Management](#tls-management)
  - [Access to Private Registry](#access-to-private-registry)
  - [Multiple Schedulers](#multiple-schedulers)
  - [Cluster Troubleshooting](#cluster-troubleshooting)
    - [Application Failure](#application-failure)
    - [Control Plane Failure](#control-plane-failure)
    - [Worker Node Failure](#worker-node-failure)
    - [Network Failure](#network-failure)
- [VPA](#vpa)
- [Networking](#networking)
  - [CoreDNS](#coredns)
  - [Network Namespaces](#network-namespaces)
  - [Docker Networking](#docker-networking)
  - [Weave Setup](#weave-setup)
  - [Calico Setup](#calico-setup)
  - [Ingress Controller - NGINX Gateway Fabric](#ingress-controller---nginx-gateway-fabric)
- [Authentication](#authentication)
  - [Kubeconfig](#kubeconfig)
  - [Kubectx and Kubens](#kubectx-and-kubens)
- [Authorization](#authorization)
- [API Maintenance](#api-maintenance)
  - [Admission Controllers](#admission-controllers)
  - [Dynamic Admission Controller](#dynamic-admission-controller)
    - [Validating / Mutating Webhooks](#validating--mutating-webhooks)
- [Objects](#objects)
  - [Selection and Export](#selection-and-export)
  - [Creation, Deletion](#creation-deletion)
  - [Replace, Modify, Scale](#replace-modify-scale)
- [Rollout, Updates](#rollout-updates)
- [Port Forwarding](#port-forwarding)
- [Helm](#helm)

<!-- /code_chunk_output -->

---

## System Architecture

### Common Linux paths for Kubernetes nodes

#### Controller node (kubeadm layout)

```bash
tree -L 2 /etc/kubernetes /etc/systemd/system/kubelet.service.d /var/lib/kubelet /var/lib/etcd /etc/cni/net.d /opt/cni/bin

/etc/kubernetes                       # kubeadm control plane configuration and kubeconfigs
├── admin.conf                        # admin client kubeconfig for cluster administration
├── controller-manager.conf           # kubeconfig used by kube-controller-manager
├── front-proxy-client.conf           # kubeconfig for aggrefgated API server front-proxy
├── kubelet.conf                      # kubeconfig used by the local kubelet to authenticate to API server
├── scheduler.conf                    # kubeconfig used by kube-scheduler
├── manifests                         # static pod manifests for control plane components
│   ├── kube-apiserver.yaml           # API server static pod manifest
│   ├── kube-controller-manager.yaml  # controller manager static pod manifest
│   └── kube-scheduler.yaml           # scheduler static pod manifest
└── pki                               # certificate authority and component TLS keys/certs
    ├── ca.crt                        # cluster CA certificate
    ├── ca.key                        # cluster CA private key
    ├── sa.key                        # service account signing key
    ├── sa.pub                        # service account public key
    ├── apiserver.crt                 # API server TLS certificate
    ├── apiserver.key                 # API server TLS private key
    ├── apiserver-kubelet-client.crt  # kube-apiserver client cert for kubelet
    └── apiserver-kubelet-client.key  # kube-apiserver client key for kubelet

/etc/systemd/system/kubelet.service.d  # kubelet systemd drop-in configuration
└── 10-kubeadm.conf                    # kubeadm-managed kubelet service settings

/var/lib/kubelet                      # kubelet runtime state and configuration
├── config.yaml                       # kubelet configuration file
├── pods                              # runtime pod directory managed by kubelet
└── pki                               # kubelet client certificates and CA trust

/etc/cni/net.d                        # CNI network configuration directory
└── 10-bridge.conf                    # example CNI network config used by the node

/opt/cni/bin                        # CNI plugin binaries installed on the node
├── bridge                          # bridge plugin for pod networking
├── host-local                      # CNI IPAM plugin for local address assignment
├── ipvlan                          # ipvlan plugin for L2/L3 pod networking
└── loopback                        # CNI loopback plugin required by spec

/var/lib/etcd                       # etcd data directory for the control plane
└── member                          # etcd member storage
    ├── wal                         # etcd write-ahead log files
    └── snap                        # etcd database snapshots
```

#### Worker node (kubeadm layout)

```bash
tree -L 2 /etc/kubernetes /etc/systemd/system/kubelet.service.d /var/lib/kubelet /etc/cni/net.d /opt/cni/bin
/etc/kubernetes                      # kubeadm kubelet configuration and certificates
├── kubelet.conf                     # kubeconfig used by kubelet to connect to the API server
└── pki                              # kubelet client certificates and CA trust
    ├── kubelet.crt                  # kubelet client certificate
    ├── kubelet.key                  # kubelet client private key
    └── ca.crt                       # cluster CA certificate trusted by kubelet

/etc/systemd/system/kubelet.service.d  # kubelet systemd drop-in configuration
└── 10-kubeadm.conf                    # kubeadm-managed kubelet service overrides

/var/lib/kubelet                    # kubelet runtime state, pods, and credentials
├── config.yaml                     # kubelet runtime config file
├── pods                            # runtime pod directories created by kubelet
└── pki                             # kubelet certificate/key storage

/etc/cni/net.d                      # CNI network configuration directory
└── 10-bridge.conf                  # CNI config used by the node

/opt/cni/bin                        # CNI plugin binaries installed on the node
├── bridge                          # bridge plugin for pod networking
├── host-local                      # local IPAM plugin for address allocation
├── ipvlan                          # ipvlan plugin for layer 2/3 pod networking
└── loopback                        # CNI loopback plugin required by the CNI spec
```

> Note: other popular Kubernetes distributions use different root paths, e.g. `k3s` under `/etc/rancher/k3s` and `microk8s` under `/var/snap/microk8s/current`.

---

## Docker

volume mount options:

```bash
# volume mount
docker run \
  --mount type=bind,source=/data/mysql,target=/var/lib/mysql \
  mysql

# use custom volume driver
docker run -it \
  --name mysql \
  --volume-driver rexray/ebs \
  --mount src=ebs-vol,target=/var/lib/mysql \
  mysql
```

## Cluster Maintenance

### Component and Node Management

**get control/data plane pods**

```bash
# get etcd pods
kubectl get pods -n kube-system | grep etcd
                                  grep kube-apiserver
                                  grep controller-manager
                                  grep scheduler
                                  grep kubelet
                                  grep kube-proxy

# get kube-apiserver pods
kubectl get pods -n kube-system -l component=etcd
                                -l component=kube-apiserver
                                -l component=kube-controller-manager
                                -l component=kube-scheduler
                                -l component=kubelet
                                -l component=kube-proxy
```

**node management**

```bash
kubectl get nodes # list nodes and kubelet versions

kubectl node cordon <node-name>      # mark node as unschedulable
kubectl node uncordon <node-name>    # mark node as schedulable
kubectl node drain <node-name>       # evict pods and mark node as unschedulable
```

### Cluster Upgrade

**upgrade commands**

```bash
### kubeadm/kubelet upgrade (controlplane+node) -- upgrade kubeadm (apt)

vim /etc/apt/sources.list
vim /etc/apt/sources.list.d/kubernetes.list
apt update
apt-cache madison kubeadm/kubelet
apt-get install kubeadm/kubelet=1.35.4-1.1

# restart kubelet to pick up new version
systemctl daemon-reload   # for: kubelet
systemctl restart kubelet # for: kubelet

# kubeadm upgrade (controlplane+node) -- upgrade kubeadm (dnf)
dnf update kubeadm=<version>    # upgrade kubeadm on control plane node

### controlplane node(s) upgrade

kubectl cordon controlplane
kubectl cordon controlplane     # if: controlplane is schedulable
# update kubeadm: see above
# update kubelet: see above
kubeadm upgrade plan            # see available versions and upgrade plan
kubeadm upgrade apply <version> # output of: "kubeadm upgrade plan"
kubectl uncordon controlplane

### worker node(s) upgrade

kubectl cordon <node-name>    # on: controlplane
kubectl drain <node-name>     # on: controlplane
# update kubelet: see above   # on: node
# update kubelet: see above   # on: node
kubeadm upgrade node          # on: node
# or run upgrade cmd below:
kubeadm upgrade node config --kubelet-version <version> # on: node
kubectl uncordon <node-name> # on: controlplane
```

### ETCD

**configure `etcdctl`**

```bash
# set API version (if not set defaults to v2)
export ETCDCTL_API=3

# set connection parameters
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.cr
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/peer.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/peer.key
export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379

# etcdctl CLI options
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/peer.crt \
--key=/etc/kubernetes/pki/etcd/peer.key \
--endpoints=https://127.0.0.1:2379

# example:
# list keys used by k8s w/ options
kubectl exec etcd-master -n kube-system -- \
  sh -c "ETCDCTL_API=3 \
    etcdctl get / --prefix --keys-only --limit=100 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/peer.crt \
    --key=/etc/kubernetes/pki/etcd/peer.key"
```

**get/put etcd data**

```bash
# version 3 API
etcdctl put key1 value1
etcdctl get key1

# list keys used by k8s
kubectl exec etcd-master -n kube-system -- \
  etcdctl get / --prefix --keys-only
```

### Backup & Restore

**backup and restore resources**

```bash
# backup all resource definitions
kubectl get all --all-namespaces -o yaml > all-deployments-backup.yaml
```

**backup/restore etcd w/ `etcdutl`**

```bash
# etcd must be stopped for file-based backup/restore
# raw file-level backup of etcd data and WAL files
etcdutl backup  \
  --data-dir=/var/lib/etcd \
  --backup-dir=/var/lib/etcd-backup

# etcd restore
cp /var/lib/etcd-backup /var/lib/etcd
```

**backup/restore etcd w/ `etcdctl`**

```bash
# etcd must be running for snapshot-based backup/restore
# snapshot-based backup
etcdctl snapshot save path/to/snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/peer.crt \
  --key=/etc/kubernetes/pki/etcd/peer.key
etcdctl snapshot status path/to/snapshot.db \
  --write-out-table # check snapshot file info/metadata

# etcd restore
systemctl stop kube-apiserver
etcdutl snapshot restore path/to/snapshot.db --data-dir /var/lib/etcd-from-backup
# adjust etcd service definition to point to new data dir
systemctl daemon-reload
systemctl restart etcd
systemctl start kube-apiserver
```

### TLS Setup

**view certificate details:**

```bash
openssl x509 -in /etc/kubernetes/manifests/pki/apiserver.crt -text -noout

# service setup
journalctl -u etcd.service -l

# kubeadm/pod setup
kubectl logs etcd-master
crictl ps -a
crictl logs <container-id>
```

**generate `CA` keys and certificates**

```bash
# create ca.key
openssl genrsa -out ca.key 2048

# create ca.csr
openssl req -new -key -subj "/CN=KUBERNETES-CA" -out ca.csr

# self-sign ca.crt
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

**generate `admin` keys and certificates** (follow same logic for other client/server component certs)

```bash
# create admin.key
openssl genrsa -out admin.key 2048

# create admin.csr
openssl req -new -key admin.key -subj "/CN=kube-admin/OU=system:masters" -out admin.csr

# sign admin.crt using CA
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

**kube-api openssl config file**

pass it to `.csr` use:

```bash
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" \
  -out apiserver.csr -config openssl.cnf
```

to **sign `.csr`** run:

```bash
openssl x509 -req -in apiserver.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -extensions v3_req -extfile openssl.cnf \
  -out apiserver.crt -days 1000
```

```ini
# openssl.cnf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name

[req_distinguished_name]
countryName = Country Name (2 letter code)
stateOrProvinceName = State or Province Name (full name)
localityName = Locality Name (eg, city)
organizationalUnitName = Organizational Unit Name (eg, section)
commonName = Common Name (eg, YOUR name)
emailAddress = Email Address

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.1 = 172.17.0.87
```

**use `admin` cert + key** in requests

```bash
curl https://kube-apiserver:64433/api/v1/pods \
  --key admin.key --cert admin.crt \
  --cacert ca.crt
```

### TLS Management

**issue certificates through API**

```bash
# generate .key and .csr
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr

# define CSR yaml
cat jane.csr ∣ base64
kubectl create -f jane.yaml
```

`jane.yaml`:

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
    - system:authenticated
  usages:
    - digital signature
    - key encipherment
    - server auth
    - client auth
  request: <base64-csr-goes-here>
```

**manage certificate signing by admin**

```bash
kubectl get csr
        certificate approve jane
        get csr jane -o yaml # .status.certificate
```

### Access to Private Registry

**create a `docker-registry` secret object**

```bash
kubectl create secret docker-registry regcred \
    --docker-server=private-registry.io \
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry-user@org.com
```

**specify the secret in pod definition file**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
  imagePullSecrets:
    - name: regcred
```

### Multiple Schedulers

see: architecture > control plane > scheduler

implementation steps:

1. create `KubeSchedulerConfiguration` config file
2. start scheduler binary/pod using config file

### Cluster Troubleshooting

- `kubectl get/describe`
- logs (pods `kubectl logs`)
- logs (services `systmectl/service/journalctl`)
- node status `top`
- certs (issuer/expire date/group)
  `openssl x509 -in /var/lib/kbelet/worker-1.crt-text | grep -Ei 'issuer:|not after:|subject:'`
- dns `nslookup`
- iptables/ipvs `iptables/ipvsadm`

#### Application Failure

1. ingress
2. service
3. pod(web)
4. pod(db)

#### Control Plane Failure

1. node status
2. pod status
3. control plane components status (pods/services)
4. control plane components logs (pods/services)

#### Worker Node Failure

1. node status
2. worker nodes components status (pods/services)
3. worker nodes components logs (pods/services)
4. worker nodes certs

#### Network Failure

**Network components:**

- pod (unique IPs)
- services (stable IP)
- coreDNS see: [[networking]] > CoreDNS
  - deployment
    - pods
  - configmap
  - service
  - serviceaccount
  - clusterRole/clusterRoleBinding
- CNI plugins (setup IPs, configure net-ifs)
- kube-proxy (rule managament)
  service-to-pod proxying using iptables/IPVS networking rules

**Troubleshooting steps:**

- **check pod/service issues**
  1. check pod status/IPs
     `kubectl get pods all -o=jsonpath='{.items[*].status.podIP}'`
  2. check if pods reachable via IP:port
     telnet/`wget -qO- $IP:$PORT`
  3. check service definition/endpoints
  - selector-labels/ports
  - check if service has (pod) endpoints
    `kubectl get endpoints -l kubernetes.io/service-name=my-service -n default`

- **check DNS (coreDNS)**
  1. check pods
     1. check status/logs
        `kubectl get pods -n kube-system -l k8s-app=kube-dns`
        `kubectl logs -n kube-system -l k8s-app=kube-dns`
     2. check endpoints
        `kubectl get endpoints -l k8s.io/service-name=kube-dns -n kube-system`
  2. check app pod config
     `kubectl exec -it my_pod -- cat /etc/resolv.conf`
     configs:
     - coredns service cluster ip
     - search path
     - ndots: 5
  3. check app pod connectivity
     `kubectl exec -it busybox -- nslookup kubernetes.default.svc.cluster.local`
     `kubectl exec -it busybox -- nslookup app-service.default.svc.cluster.local`

- **check CNI plugins**
  check CNI deamonset pod status/logs

- **kube-proxy**
  1. check kube-proxy status/logs (pods/service)
  2. check settings configmap
     - mode (ipvs/ipdtables)
     - clusterCIDR
  3. verify iptables/ipvs network rules
     `ipvsadm -ln`

---

## VPA

**implementation steps:**

1. **create CRDs**

- `verticalpodautoscalercheckpoints`
- `verticalpodautoscalers`

2. **define RBAC**

- `clusterrole/system:metrics-reader`
- `clusterrole/system:vpa-actor`
- `clusterrole/system:vpa-status-actor`
- `clusterrole/system:vpa-checkpoint-actor`
- `clusterrole/system:evictioner`
- `clusterrolebinding/system:metrics-reader`
- `clusterrolebinding/system:vpa-actor`
- `clusterrolebinding/system:vpa-status-actor`
- `clusterrolebinding/system:vpa-checkpoint-actor`
- `clusterrole/system:vpa-target-reader`
- `clusterrolebinding/system:vpa-target-reader-binding`
- `clusterrolebinding/system:vpa-evictioner-binding`
- `serviceaccount/vpa-admission-controller`
- `serviceaccount/vpa-recommender`
- `serviceaccount/vpa-updater`
- `clusterrole/system:vpa-admission-controller`
- `clusterrolebinding/system:vpa-admission-controller`
- `clusterrole/system:vpa-status-reader`
- `clusterrolebinding/system:vpa-status-reader-binding`

3. **clone VPA repository**
   `git clone https://github.com/kubernetes/autoscaler.git`

4. **run setup script**
   `./autoscaler/vertical-pod-autoscaler/hack/vpa-up.sh`

   _script output:_

   ```
   HEAD is now at 9196162ba Update VPA default version to 1.6.0
   customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalercheckpoints.autoscaling.k8s.io configured
   customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalers.autoscaling.k8s.io configured
   clusterrole.rbac.authorization.k8s.io/system:metrics-reader unchanged
   clusterrole.rbac.authorization.k8s.io/system:vpa-actor configured
   clusterrole.rbac.authorization.k8s.io/system:vpa-status-actor unchanged
   clusterrole.rbac.authorization.k8s.io/system:vpa-checkpoint-actor unchanged
   clusterrole.rbac.authorization.k8s.io/system:evictioner unchanged
   clusterrole.rbac.authorization.k8s.io/system:vpa-updater-in-place created
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-updater-in-place-binding created
   clusterrolebinding.rbac.authorization.k8s.io/system:metrics-reader unchanged
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-actor unchanged
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-actor unchanged
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-checkpoint-actor unchanged
   clusterrole.rbac.authorization.k8s.io/system:vpa-target-reader unchanged
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-target-reader-binding unchanged
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-evictioner-binding unchanged
   serviceaccount/vpa-admission-controller unchanged
   serviceaccount/vpa-recommender unchanged
   serviceaccount/vpa-updater unchanged
   clusterrole.rbac.authorization.k8s.io/system:vpa-admission-controller configured
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-admission-controller unchanged
   clusterrole.rbac.authorization.k8s.io/system:vpa-status-reader unchanged
   clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-reader-binding unchanged
   role.rbac.authorization.k8s.io/system:leader-locking-vpa-updater created
   rolebinding.rbac.authorization.k8s.io/system:leader-locking-vpa-updater created
   role.rbac.authorization.k8s.io/system:leader-locking-vpa-recommender created
   rolebinding.rbac.authorization.k8s.io/system:leader-locking-vpa-recommender created
   deployment.apps/vpa-updater created
   deployment.apps/vpa-recommender created
   Generating certs for the VPA Admission Controller in /tmp/vpa-certs.
   Certificate request self-signature ok
   subject=CN = vpa-webhook.kube-system.svc
   Uploading certs to the cluster.
   secret/vpa-tls-certs created
   Deleting /tmp/vpa-certs.
   service/vpa-webhook created
   deployment.apps/vpa-admission-controller created
   service/vpa-webhook unchanged
   ```

## Networking

### CoreDNS

**download and run CoreDNS**

- by default, listens on port 53

```bash
wget https://github.com/coredns/coredns/releases/download/v1.7.0/coredns_1.7.0_linux_amd64.tgz
coredns_1.7.0_linux_amd64.tgz
tar -xzvf coredns_1.7.0_linux_amd64.tgz
coredns
./coredns
```

**configure CoreDNS**

- add entries into the `/etc/hosts` file
- Configure `Corefile` to use hosts entries
- start/restart/reload CoreDNS

```properties
.53: {
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

### Network Namespaces

**setup virtual/bridge network** (uses Linux Bridge)

```bash
#------------------------------------
### create netns and bridge network

# create a network namespace
ip netns add red
ip netns add blue

# add a new interface to the host to create a internal virt-network
ip link add v-net-0 type bridge
ip link set dev v-net-0 up # turn the interface up

#------------------------------------
### link netns and virt-network using virt-cable

# create a virt-cable
ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name veth-blue-br

# connect virt-cable ends to netns and the virt-network
ip link set veth-red netns red
ip link set veth-blue netns blue
ip link set veth-red-br master v-net-0
ip link set veth-blue-br master v-net-0

# add IP addr
ip -n red addr add 192.168.15.1/24 dev veth-red   # virt-cable netns interfaces
ip -n blue addr add 192.168.15.2/24 dev veth-blue # virt-cable netns interfaces
ip addr add 192.168.15.5/24 dev v-net-0           # virt-network interface (ip link)

# activate virt-cable interfaces
ip -n red link set veth-red up
ip -n blue link set veth-blue up

# activate virt-cable interfaces (attached to host/virt-switch link)
ip link set dev veth-red-br up
ip link set dev veth-blue-br up

#------------------------------------
### configure egress communication for netns

# configure routes on netns
ip netns exec blue route
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
ip netns exec blue ip route add default via 192.168.15.5

# enable NAT for packets routing from virt-network on host (for destination response)
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
# -t    Use the NAT table
# -A    Append rule to POSTROUTING chain (rules applied after routing decision)
# -s    Match source IP addresses from virtual network subnet
# -j    "Jump" to MASQUERADE target which replaces source IP with host's IP

#------------------------------------
### configure ingress communication to netns

# add port forwarding rule to allow external traffic to reach container
iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
# -t nat              Use NAT table for port forwarding rules
# -A PREROUTING       Add rule to PREROUTING chain (rules applied before routing decision)
# --dport 80          Match destination port 80 (HTTP)
# --to-destination    Forward matched traffic to container IP and port
# -j                  Jump to DNAT target to modify destination address
```

display NAT table rules

```bash
iptables -nvL -t nat
# -n         Show numeric output (don't resolve hostnames)
# -v         Verbose output
# -L         List rules
# -t nat     Show NAT table
```

### Docker Networking

see interfaces

```bash
docker run nginx

# check netns
ip netns
ip -n $(ip netns) link  # check virt-cable netns's endpoint
ip -n $(ip netns) addr  # check virt-cable netns's endpoint address
ip link                 # check virt-cable virt-network's bridge port(look for 'master docker0')
```

use docker w/ CNI (how kubernetes uses docker)

```bash
# create docker w/ no net config
docker run --network=none nginx

ip netns             # get netns_id
bridge add netns_id  /var/run/netns/netns_id # manually invokecni plugin
```

### Weave Setup

```yaml
# deployed as deamonset pods on the cluster
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

### Calico Setup

see: self-managed on-premises installation guide:
https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

```bash
# 01. Install the Tigera Operator and custom resource definitions
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/v1_crd_projectcalico_org.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/tigera-operator.yaml

# 02. Download the custom resources necessary to configure Calico
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/custom-resources.yaml

# 03. customize operator (cidr, etc.) and apply
vim custom-resources.yaml
kubectl create -f custom-resources.yaml

# 04. Monitor the deployment
watch kubectl get pods -n calico-system
watch kubectl get tigerastatus

```

### Ingress Controller - NGINX Gateway Fabric


1. install an nginx gateway API

```bash
# Install the Gateway API resources
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | kubectl apply -f -
# If the GitHub-based kustomize fetch times out, use the local tarball fallback below.
mkdir -p /tmp/ngf
curl -L --fail https://codeload.github.com/nginx/nginx-gateway-fabric/tar.gz/refs/tags/v1.5.1 \
  | tar -xz -C /tmp/ngf --strip-components=1
kubectl kustomize /tmp/ngf/config/crd/gateway-api/standard | kubectl apply -f -


# Deploy the NGINX Gateway Fabric CRDs
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml

# Deploy NGINX Gateway Fabric
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml

# Verify the Deployment
kubectl get pods -n nginx-gateway

# View the nginx-gateway service
kubectl get svc -n nginx-gateway nginx-gateway -o yaml

# Update the nginx-gateway service to expose ports 30080 for HTTP and 30081 for HTTPS
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'

```

2. create gateway-listener > route

see: [labs/gateway-nginx]

---

## Authentication

### Kubeconfig

kubectl global options

```bash
kubectl options
```

kube config

```bash
kubectl config -h
kubectl config view
kubectl config view --kubeconfig=/path/to/kubeconfig
kubectl config use-context kubeadmin@kubeplayground
```

use `kubectl proxy`

```bash
# reads kubeconfig and adds auth conf to commands
kubectl proxy
  Starting to serve on 127.0.0.1:8001
```

### Kubectx and Kubens

`kubectx`: switch between contexts easily

```bash
kubectx                  # list all contexts
kubectx -c               # list current context
kubectx <context_name>   # switch to context
kubectx -                # switch to last context
```

`kubens`: switch between namespaces easily

```bash
kubens <namespace>  # switch to namespace
kubens -            # switch to last namespace
```

## Authorization

**describe roles and rolebindings**

```bash
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding
```

**check user access**

```bash
kubectl auth can-i [--as <user-name>] <verb> <resource>

kubectl auth can-i [--as dev-user] create deployments
kubectl auth can-i [--as dev-user] delete pods
```

view **enabled admission controllers**

```bash
kube-apiserver -h | grep enable-admission-plugin

# in a kubeadm setup, run in kube apiserver controlplane pod
kubectl exec kube-apiserver-controlplane -n kube-system -- \
  kube-apiserver -h | grep enable-admission-plugin
```

## API Maintenance

**explore k8s objects**

```bash
# get api preferred versions on the server as "group/preferred-version"
kubectl api-versions

# get api resources
kubectl api-resources [--namespaced=[ true|false ]]

# get resource structure
kubectl explain <resource>[.<field-name>] [--recursive [ true|false ]]
```

**discover API tree**

```bash
# bypass required auth (uses creds in kubeconfig)
kubectl proxy # serves on localhost:8001

curl https://api-server-url:8001 -k # API endpoints under "paths"
curl https://api-server-url:8001/apis -k | grep "name"
curl https://my-kube-playground:6443/version
curl https://my-kube-playground:6443/api/v1/pods
```

see **api-group preferred version:**

```bash
curl 127.0.0.1:8001/apis/batch | grep -iA5 preferredversion
```

see **storage version**

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=<path> \
  --cert=<path> \
  --key=<path> \
  get "/registry/deployment/default/<deployment-name> --print-value-only
```

**enable/disable API group**

```bash
ExecStart=/usr/local/bin/kube-apiserver \\
  ...
  --runtime-config=batch/v2alpha1,... \\
  ...
```

bulk **convert definition files** form a version to another

```bash
# need to install the convert plugin
kubectl convert -f nginx_def.yaml --output-version <new-api>
```

### Admission Controllers

```bash
# see default controllers
k exec kube-apiserver-controlplane -n kube-system -- \
  kube-apiserver -h | grep -i enable-admission-plugins

# see current controllers besides default
ps aux | grep -v grep | grep -i kube-apiserver | grep -i admission

# add/remove admission controller
# valid for kubeadm deployments
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Dynamic Admission Controller

see: architecture > control-plane > controllers > dynamic_admission_controller

Admission controller setup steps:

1. create tls secret for webhook deployment
2. create webhook deployment (uses tls secret)
   lab image: `stackrox/admission-controller-webhook-demo:latest`
3. create service (expose webhook)

#### Validating / Mutating Webhooks

- create webhook configuration (webhook config in api-server):
  - ValidatingWebhookConfiguration
  - MutatingWebhookConfiguration.

---

## Objects

### Selection and Export

object selection (not always interchangeable)

```bash
[ all|<resource-type> ]           # all objects
[ <type> <name>|<type>/<name> ]   # type and name selection
-f resource_definition.yaml         # file def selection
```

namespace selection

```bash
# namespace selection
kubectl [command] [object] [ -A|--all-namespaces ]
kubectl [command] [object] [[ -n|--namespace ] <namespace-name>]

# switch to namespace
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

get resource definition _output_format: name, wide, yaml, json_

```bash
# objects
kubectl describe <resource>
kubectl get <resource>
kubectl get <resource> [ -o <output-format> ]
  # output-format: name, wide, yaml, json

# JSON query
kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'

kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}'
kubectl get nodes -o=jsonpath='{range.items[*]} {.metadata.name} {"\t"} {.status.capacity.cpu} {"\n"} {end}'
kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu --sort-by=.status.capacity.cpu

```

get service url

```bash
minikube service <service> --url
```

get a container's logs

```bash
# <container-name> is necessary for multi-container pods.
kubectl logs -f <pod-name> [<container-name>]
```

---

### Creation, Deletion

run image on cluster

```bash
kubectl run pod_name --image=image_name
```

create resource from file or stdin

```bash
kubectl create -f file_path
```

delete resource

```bash
kubectl delete [resource]
```

configMap

```bash
kubectl create configmap <config-name> \
  --from-literal=APP_COLOR=red \
  --from-literal=APP_MOD=prod
```

secret

```bash
kubectl create secret generic <secret-name> \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Pwd=password
```

node taint

```bash
kubcetl taint nodes <node-name> key=value:<taint-effect>
  # taint-effect:
    # NoSchedule: pods will not be scheduled on the node
    # PreferNoSchedule: try to avoid placing pod on node
    # NoExecute: new pods will not be scheduled on the node, existing pods that don't tolerate the taint are evicted

kubcetl taint nodes node01 app=blue:NoSchedule
kubcetl taint nodes node01 app:NoSchedule-      # removes taint
kubcetl taint nodes node01 app=blue:NoSchedule- # removes taint
```

node labels (for node affinity)

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

---

### Replace, Modify, Scale

replace a resource

```bash
kubectl replace [resource]
```

apply config to resource

```bash
kubectl apply -f [config_file]
```

edit resource (opens editor to runtime config)

```bash
kubectl edit [resource]
```

scale resource

for a _deployment_, _replica set_, _replication controller_, or _stateful set_

```bash
kubectl scale --replicas=3 [resource]
```

change application resources (_changes are to runtime config_)

```bash
kubectl set [resource] [deployment] [container_name]=[new_image_name]
  # resources:
    env              Update environment variables on a pod template
    image            Update the image of a pod template
    resources        Update resource requests/limits on objects with pod templates
    selector         Set the selector on a resource
    serviceaccount   Update the service account of a resource
    subject          Update the user, group, or service account in a role binding or cluster role binding
```

---

## Rollout, Updates

controll rollouts

rollout is valid for _deployments_, _daemonsets_, or _statefulsets_

```bash
kubectl rollout [command] [deployment]
  # commands
    history       View rollout history
    pause         Mark the provided resource as paused
    restart       Restart a resource
    resume        Resume a paused resource
    status        Show the status of the rollout
    undo          Undo a previous rollout
```

---

## Port Forwarding

Environment

```bash
kubectl get service mongo
# NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
# mongo   ClusterIP   10.96.41.183   <none>        27017/TCP   11s

kubectl get pods
# NAME                     READY   STATUS    RESTARTS   AGE
# mongo-75f59d57f4-4nd6q   1/1     Running   0          2m4s

kubectl get pod mongo-75f59d57f4-4nd6q --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
# 27017
```

> Note: `27017` is the TCP port allocated to `mongo` pod

Forward a local port to a port on the Pod

```bash
kubectl port-forward  mongo-75f59d57f4-4nd6q       28015:27017
                      pods mongo-75f59d57f4-4nd6q  28015:27017
                      deployment mongo             28015:27017
                      replicaset mongo-75f59d57f4  28015:27017
                      service mongo                28015:27017

# Forwarding from 127.0.0.1:28015 -> 27017
# Forwarding from [::1]:28015 -> 27017
```

let kubectl choose the local port

```bash
kubectl port-forward deployment/mongo :27017
# Forwarding from 127.0.0.1:63753 -> 27017
# Forwarding from [::1]:63753 -> 27017
```

---

## Helm

install helm: https://helm.sh/docs/intro/install

```bash
# get helm client env information
helm env

# configure repos
helm repo [ add|index|list|remove|update ] [ OPTIONS ]

# search for a chart
helm search [ hub|repo ] <chart-name>

# download chart
helm pull [ <chartURL>|<repo>/<chartname> ] [ --untar ] [ --destination <path> ]

# install chart
helm install <chart-name> <repo>/<chartname>

helm list
helm status <chart-release-name>
helm uninstall <chart-release-name>

# debug templates
# verify chart follows best practices
helm lint

# test render templates locally
helm template --debug

# render templates, then return resulting manifest files
helm install --dry-run --debug

# see what templates are installed on server
helm get manifest
```
