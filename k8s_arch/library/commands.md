# K8S Commands

tags: #tools_utils

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Docker](#docker)
- [Cluster Maintenance](#cluster-maintenance)
  - [Component and Node Management](#component-and-node-management)
  - [Cluster Upgrade](#cluster-upgrade)
  - [ETCD](#etcd)
  - [Backup & Restore](#backup--restore)
  - [TLS Setup](#tls-setup)
  - [TLS Management](#tls-management)
  - [Access to Private Registry](#access-to-private-registry)
- [Authentication](#authentication)
  - [Kubeconfig](#kubeconfig)
  - [Kubectx and Kubens](#kubectx-and-kubens)
- [Authorization](#authorization)
- [API Maintenance](#api-maintenance)
- [Objects](#objects)
  - [Selection](#selection)
  - [Monitoring, Export](#monitoring-export)
  - [Creation, Deletion](#creation-deletion)
  - [Replace, Modify, Scale](#replace-modify-scale)
- [Rollout, Updates](#rollout-updates)
- [Port Forwarding](#port-forwarding)
- [Data Plane Components](#data-plane-components)
- [Admission Controllers](#admission-controllers)
- [Helm](#helm)

<!-- /code_chunk_output -->

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
# control node upgrade
dnf update kubeadm=<version>   # upgrade kubeadm on control plane node
kubeadm upgrade plan
kubeadm upgrade apply <version>
systemctl restart kubelet      # restart kubelet to pick up new version

# worker node upgrade
kubectl drain <node-name> --ignore-daemonsets # master
dnf update kubeadm=<version> kubectl=<version>
kubeadm upgrade node config --kubelet-version <version>
systemctl restart kubelet
kubectl uncordon <node-name> # master
```

### ETCD

**configure etcd client**

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
  request: <certificate-goes-here>
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

---

## Objects

### Selection

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

---

### Monitoring, Export

export resource definition file. _output_format: name, wide, yaml, json_

```bash
# objects
kubectl describe <resource>
kubectl get <resource>
kubectl get <resource> [ -o <output-format> ]
  # output-format: name, wide, yaml, json
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

## Data Plane Components

## Admission Controllers

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
