# K8S User Auhorization

tags: #security

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S User Auhorization](#k8s-user-auhorization)
  - [RBAC](#rbac)
    - [Roles and RoleBindings](#roles-and-rolebindings)
    - [ClusterRoles and ClusterRoleBindings](#clusterroles-and-clusterrolebindings)
  - [AdmissionController](#admissioncontroller)
    - [Dynamic AdmissionController](#dynamic-admissioncontroller)

<!-- /code_chunk_output -->

---

Authorization modes

- **AlwaysAllow** # default
  allows all requests w/out auth checks

- **AlwaysDeny**
  denies all requests w/out auth checks

- **Node Authorizer**
  for internal cluster access, ex: kubelets must

  - be part of the system `nodes` group
  - have a name prefix `system-node`

- **ABAC** (Atribute Based Access Control)

  - implemented with a policy file.
  - requires apiServer restart with each file modification

        {"kind": "Policy, "spec": {"user": "dev-user", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
        {"kind": "Policy, "spec": {"user": "dev-user-2", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
        {"kind": "Policy, "spec": {"user": "dev-user-group", "namespace": "*", "resource": "pods", "apiGroup": "*"}}

- **RBAC** (Role Based Access Control)

  - associates `users` and `groups` with `roles`

- **Webhook**

  - external auth tools
  - accessed using API calls

specify auth mode to `kube-apiserver`

    # attmpted auth order: Node > RBAC > Webhook
    /usr/local/bin/kube-apiserver \
      ...
      --authorization-mode=Node,RBAC,Webhook
      ...

## RBAC

Steps:

- create `Role`/`ClusterRole`
- create `RoleBinding`/`ClusterRoleBinding`

get user access

    kubectl auth can-i [--as dev-user] create deployments
    kubectl auth can-i [--as dev-user] delete pods

### Roles and RoleBindings

- for namespaced-scoped resources (`jobs`, `deployments`, `services`, `configmaps`, `PVC`,...)

[[role]]

[[roleBinding]]

### ClusterRoles and ClusterRoleBindings

- for cluster-scoped resources (nodes, PV, namespaces,..)
- can be created for namespaced-resources,
  - clusterRole will **apply across ALL namespaces for that resource**

[[clusterRole]]

[[clusterRoleBinding]]

## AdmissionController

[[admission_controller]]

### Dynamic AdmissionController

[[admission_controller]] > [dynamic_admission_controller]
