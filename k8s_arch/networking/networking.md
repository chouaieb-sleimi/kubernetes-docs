# K8S Networking

tags: #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Networking Concepts](#networking-concepts)
  - [Network Namespaces](#network-namespaces)
  - [Docker Networking](#docker-networking)
  - [Container Networking Interface (CNI)](#container-networking-interface-cni)
  - [Pod Networking](#pod-networking)
  - [Weave CNI + IPAM](#weave-cni--ipam)
- [Cluster Networking](#cluster-networking)
  - [NetworkPolicy](#networkpolicy)
  - [CoreDNS](#coredns)
  - [Port Forwarding](#port-forwarding)
- [Service Networking](#service-networking)
  - [NodePort](#nodeport)
  - [ClusterIP](#clusterip)
  - [LoadBalancer](#loadbalancer)
- [Ingress](#ingress)
  - [Ingress Controller](#ingress-controller)
  - [Ingress Resources](#ingress-resources)
  - [Gateway API](#gateway-api)

<!-- /code_chunk_output -->

---

- Internal Private Network is created when k8s is created/configured
- Each pod has an ip address

**K8S Fundamental Networking Requirements:**

- **Containers or PODs** in a cluster MUST be able to **communicate without configuring NAT**.

- **All nodes** must be able to **communicate with containers** in the cluster.
- **All containers** must be able to **communicate with the nodes** in the cluster.

---

## Networking Concepts

### Network Namespaces

see: commands > Network Namespaces

**setup network namespace** using pipe/virtual cable:

1. **create netns**
1. **create virtual/bridge network** (interface)
1. **create `veth` pipe/ virt-cable** w/ its 2 interfaces/ends
1. **attach `veth` interface** to netns
1. **attach other `veth` interface** to the bridge
1. **assign ip addr** to interfaces
1. **activate interfaces**
1. **enable NAT-IP masquerade** for egress

**virtual networks/switches** solutions:

- Linux Bridge
- Open vSwitch (OvS)

**setup Linux Bridge** for virtual networks

- **create network namespaces and virtual/bridge network** (host interface)
- **link netns and virt-network** using virt-cable
- **enable egress traffic from** netns
  - configure routes on netns
  - enable NAT for packets routing from virt-network on host
- **(optional) enable ingress traffic to** netns
  - **option 1:** define route in the source: `ip route add <netns-addr> via <host-addr>`
  - **option 2:** configure port forwarding on host: `ip route add <netns-addr> via <host-addr>`

> **Note:** in this setup, the host provides to netns:
>
> - gateway
> - default gateway
> - NAT

### Docker Networking

docker network types:

- **none:** unreachable to/from the ouside
- **host:** contianer is attached to host network (share host ports)
- **bridge:**
  - default network `docker0`
  - internal private network
  - nat-ed network using `docker0` bridge host interface
  - `docker0` netns `id` is present when inspecting: `containers[].NetworkSettings.SandboxID+SandboxKey`

**when a contianer is created**, docker (see netns setup above):

- creates netns
- attaches netns to bridge network
- assigns an ip addr to container interface from bridge network subnet
- **configure egress traffic:**
  - adds nat rule to iptables for nat-ing container traffic via host interface
  - configures **iptables MASQUERADE rules** for outbound traffic
  - **inside container:**
    - adds **route to outside** via host interface
    - adds **dns server addr resolv.conf** (usually host ip addr)
    - adds **gateway addr to route table** (usually bridge interface ip addr)
- **configure ingress traffic:**
  - adds route to container interface via bridge interface
  - creates DNAT rules if ports are published (-p flag)
  - updates Docker proxy rules if ports are published

**docker networking model:** Contianer Network Model (CNM)

### Container Networking Interface (CNI)

for CNI, see: https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-network-model
for addons, see: https://kubernetes.io/docs/concepts/cluster-administration/addons

- defines **container-runtime/plugin standards**
- defines **container-runtime responsibilities:**
  - runtime must create netns
  - runtime to invoke network plugin when container is added/deleted
  - runtime to invoke network plugin when container is deleted
  - defines `json` format of the netowrk config
- defines **network plugins responsibilities:**
  - must support CLI args: `ADD/DEL/CHECK`
  - must support CLI params: container id, netns, etc.
  - must manage IP addr assignment to pods
  - must return results in specific format
- **plugins:** (path: `/opt/cni/bin/`)
  - **builtin**
    - `bridge:` steps 02-08 (see netns setup above)
      - step 01 (netns creation) is done by container-runtime (docker, rkt, cri-o, etc.)
    - `vlan`
    - `ipvlan`
    - `macvlan`
    - `windows`
  - **IPAM Plugins (IP Address Management)**
    - `host-local`
    - `dhcp`
  - **3rd aprty**
    - `flannel`
      doesn't support k8s`networkPolicy`
    - `calico`
      one of the most capable
    - `weave`
    - `cilium`
    - `infoblocks`
- CNI configuration at container-runtime:
  `--cni-conf-dir=/etc/cni/net.d`
  `--cni-bin-dir=/etc/cni/bin`
  ```json
  // /etc/cni/net.d/10-bridge.conf
  {
    "cniVersion": "0.3.1",
    "name": "mybridgenet",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true, // gives bridge interface IP so that it can act as gateway
    "ipMasq": true,
    "ipam": {
      // subnets+routes to be assigned to pods
      "type": "host-local", // IPs are managed locally
      "subnet": "10.244.1.0/24",
      "routes": [{ "dst": "0.0.0.0/0" }]
    },
    "dns": {
      "nameservers": ["8.8.8.8"]
    }
  }
  ```

### Pod Networking

Networking model (challenges):

- (1) every pod has an IP
- (2) every pod should communicate to other pods **on the same node**
- (3) every pod should communicate to other pods **on other nodes** without NAT

Implementation:

- create bridge network on each node
- assign IP addr to each bridge interface
- for each container: run `net-script.sh` (CNI plugin)
  - create veth pair
  - attach veth ends to bridge and container netns
  - assing ip addr & setup default gateway routes
    => (1)
    => (2)
- add routes on each node to other networks
  **or** configure central router w/ routing table

`net-script.sh`

```bash
### CNI action: ADD
# create veth pair
ip link add ...

# attach veth pair
ip link set ...
ip link set ...

# Invoke IPAM plugin (host-local)
ip = get_free_ip_from_host_local()

# assign Ip addr
ip -n <netns> addr add ...
ip -n <netns> route add ...

# bring up interface
ip -n <netns> link set ...

### CNI action: DEL
# delete veth pair
ip lin del ...
```

Architecture:

```bash
LAN (192.168.1.0)
├── node1 (192.168.1.11)
│   └── bridge: vnet-0 (10.244.1.0/24)
│       bridge interface: 10.244.1.1
│       ├── pod1 netns: 10.244.1.2
│       └── pod2 netns: 10.244.1.2
│   - route 10.244.1.2 via 192.168.1.12
│   - route 10.244.1.3 via 192.168.1.13
│
├── node2 (192.168.1.12)
│   └── bridge: vnet-0 (10.244.2.0/24)
│       bridge interface: 10.244.2.1
│       └── pod1 netns: 10.244.2.2
│   - route 10.244.1.1 via 192.168.1.11
│   - route 10.244.1.3 via 192.168.1.13
│
├── node3 (192.168.1.13)
│   └── bridge: vnet-0 (10.244.3.0/24)
│       bridge interface: 10.244.3.1
│       └── pod1 netns: 10.244.3.2
│   - route 10.244.1.2 via 192.168.1.12
│   - route 10.244.1.1 via 192.168.1.11
│
└── router (alternative to manual routes)
    routing table:
    ├── dest=10.244.1.0/24: gateway=192.168.1.11
    ├── dest=10.244.2.0/24: gateway=192.168.1.12
    └── dest=10.244.3.0/24: gateway=192.168.1.13
```

### Weave CNI + IPAM

---

## Cluster Networking

**service DNS structure:** `web-service.apps.svc.cluster.local`

- hostname(service name): `web-service`
- sub-domain(namespace): `apps`
- domain(type): `svc`
- root: `cluster.local`

**pods DNS structure:** `10-244-2-5.apps.pod.cluster.local`

- hostname(pod IP): `10-244-2-5`
- sub-domain(namespace): `apps`
- domain(type): `pod`
- root: `cluster.local`

**cluster ports** to open (inbound):
see: https://kubernetes.io/docs/reference/networking/ports-and-protocols/

| Node Type   | Direction | Port Range          | Purpose                            | Used By              |
| ----------- | --------- | ------------------- | ---------------------------------- | -------------------- |
| **Control** | Inbound   | 6443                | Kubernetes API server              | All                  |
| **Control** | Inbound   | 2379-2380           | etcd server client API             | kube-apiserver, etcd |
| **Control** | Inbound   | 10250               | Kubelet API                        | Self, Control plane  |
| **Control** | Inbound   | 10259               | kube-scheduler                     | Self                 |
| **Control** | Inbound   | 10257               | kube-controller-manager            | Self                 |
| **Worker**  | Inbound   | 10250               | Kubelet API                        | Self, Control plane  |
| **Worker**  | Inbound   | 10256               | kube-proxy                         | Self, Load balancers |
| **Worker**  | Inbound   | 30000-32767         | NodePort Services **(deprecated)** | All                  |
| **Worker**  | Inbound   | 30000-32767 **UCP** | NodePort Services **(deprecated)** | All                  |

> All current ports are **TCP**

### NetworkPolicy

[[networkPolicy]]

### CoreDNS

[[coredns]]

### Port Forwarding

[[port_forwarding]]

---

## Service Networking

[[service]]

cluster-wide virtual objects managed by kube-proxy

### NodePort

[[service]] > nodePort

### ClusterIP

[[service]] > clusterIP

### LoadBalancer

[[service]] > loadBalancer

---

## Ingress

[[ingress]]

### Ingress Controller

- not deployed by default
- controller types/backends:
  - GCP http(s) load balancer
  - nginx
  - contour
  - ha-proxy
  - traefik
  - Istio

[[ingress]] > [ingress_controller]

### Ingress Resources

[[ingress]] > [ingress_resource]

### Gateway API

[[gatewayAPI]]
