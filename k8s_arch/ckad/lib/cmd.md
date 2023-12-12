# K8S Commands

tags: #tools_utils

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Commands](#k8s-commands)
  - [Config](#config)
  - [Auth](#auth)
  - [API Maintenance](#api-maintenance)
  - [Objects](#objects)
    - [Selection](#selection)
    - [Export](#export)
    - [Creation, Deletion](#creation-deletion)
    - [Replace, Modify, Scale](#replace-modify-scale)
  - [Rollout, Updates](#rollout-updates)
  - [Port Forwarding](#port-forwarding)
  - [Admission Controllers](#admission-controllers)
  - [Helm](#helm)

<!-- /code_chunk_output -->

---

## Config

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

---

## Auth

get user access

```bash
kubectl auth can-i [--as <user-name>] <verb> <resource>

kubectl auth can-i [--as dev-user] create deployments
kubectl auth can-i [--as dev-user] delete pods
```

view enabled admission controllers

```bash
kube-apiserver -h | grep enable-admission-plugin

# in a kubeadm setup, run in kube apiserver controlplane pod
kubectl exec kube-apiserver-controlplane -n kube-system -- \
  kube-apiserver -h | grep enable-admission-plugin
```

---

## API Maintenance

explore k8s objects

```bash
# get api preferred versions on the server as "group/preferred-version"
kubectl api-versions

# get api resources
kubectl api-resources [--namespaced=[ true|false ]]

# get resource structure
kubectl explain <resource>[.<field-name>] [--recursive [ true|false ]]
```

discover API tree

```bash
curl https://api-server-url:8001 -k
curl https://api-server-url:8001/apis -k | grep name
curl https://my-kube-playground:6443/version
curl https://my-kube-playground:6443/api/v1/pods
```

see preferred version for api group:

```bash
curl 127.0.0.1:8001/apis/batch | grep -iA5 preferredversion
```

see storage version (must have `etcdctl` installed)

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=<path> \
  --cert=<path> \
  --key=<path> \
  get "/registry/deployment/default/<deployment-name> --print-value-only
```

enable/disable API group

```bash
ExecStart=/usr/local/bin/kube-apiserver \\
  ...
  --runtime-config=batch/v2alpha1,... \\
  ...
```

bulk convert definition files form a version to another

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

### Export

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

Monitoring

get a container's logs

```bash
# <container-name> is necessary for multi-container pods.
kubectl logs -f <pod-name> [<container-name>]
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

> Note: `27017` is the TCP port allocated to `mongo` pod_

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
```
