# K8S User Auhorization

tags: #security

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S User Auhorization](#k8s-user-auhorization)
  - [Authorization modes](#authorization-modes)
    - [Node Authorizer](#node-authorizer)
    - [ABAC - Atribute Based Access Control](#abac---atribute-based-access-control)
    - [RBAC - Role Based Access Control](#rbac---role-based-access-control)
    - [Webhook](#webhook)
  - [RBAC](#rbac)
    - [Roles and RoleBindings](#roles-and-rolebindings)
    - [ClusterRoles and ClusterRoleBindings](#clusterroles-and-clusterrolebindings)

<!-- /code_chunk_output -->

---

## Authorization modes

specify auth mode to `kube-apiserver`, whenever a mode/module denies a request it is forwarded to the next part of the chain

```bash
# attmpted auth order: Node > RBAC > Webhook
/usr/local/bin/kube-apiserver \
  ...
  --authorization-mode=Node,RBAC,Webhook
  ...
```


- **AlwaysAllow** # default
  allows all requests w/out auth checks

- **AlwaysDeny**
  denies all requests w/out auth checks

### Node Authorizer

for internal cluster access, ex: kubelets must be part of the system `nodes` group (name prefix `system:node`)

### ABAC - Atribute Based Access Control

- implemented with a policy file.
- requires apiServer restart with each file modification

```json
{
  "kind": "Policy",
  "spec": {
    "user": "dev-user",
    "namespace": "*",
    "resource": "pods",
    "apiGroup": "*"
  }
}
```

```json
{
  "kind": "Policy",
  "spec": {
    "user": "dev-user-2",
    "namespace": "*",
    "resource": "pods",
    "apiGroup": "*"
  }
}
```

```json
{
  "kind": "Policy",
  "spec": {
    "user": "dev-user-group",
    "namespace": "*",
    "resource": "pods",
    "apiGroup": "*"
  }
}
```

### RBAC - Role Based Access Control

- associates `users` and `groups` with `roles`

### Webhook

- external auth tools
- accessed using API calls

## RBAC

Steps:

- create `Role`/`ClusterRole`
- create `RoleBinding`/`ClusterRoleBinding`

get user access

    kubectl auth can-i [--as dev-user] create deployments
    kubectl auth can-i [--as dev-user] delete pods

### Roles and RoleBindings

[[role]]

[[roleBinding]]

- for namespaced-scoped resources (`jobs`, `deployments`, `services`, `configmaps`, `PVC`,...)

### ClusterRoles and ClusterRoleBindings

[[clusterRole]]

[[clusterRoleBinding]]

- for cluster-scoped resources (nodes, PV, namespaces,..)
- can be created for namespaced-resources,
  - clusterRole will **apply across ALL namespaces for that resource**
