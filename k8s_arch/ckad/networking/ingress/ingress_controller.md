# K8S Ingress Controller Deployment

tags: #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Ingress Controller Deployment](#k8s-ingress-controller-deployment)
  - [Ingress ConfigMap](#ingress-configmap)
  - [Ingress ServiceAccount](#ingress-serviceaccount)
    - [ServiceAccount Roles](#serviceaccount-roles)
    - [ServiceAccount RoleBindings](#serviceaccount-rolebindings)
    - [ServiceAccount ClusterRoleBindings](#serviceaccount-clusterrolebindings)
  - [Ingress Controller Deployment](#ingress-controller-deployment)
    - [Controller Deployment](#controller-deployment)
  - [Ingress Service](#ingress-service)

<!-- /code_chunk_output -->

---

**Deployment steps:**

- Create ingress **ConfigMap**
- Prepare ingress **ServiceAccount** (for monitoring ingress resource changes)
- Create controller **Deployment**
- Create ingress **Service**

## Ingress ConfigMap

create ingress configMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
```

## Ingress ServiceAccount

**prepare ingress service account**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```

> Note: Roles, ClusterRoles and RoleBindings must be configured as well

### ServiceAccount Roles

```bash
kubectl get role --namespace ingress-nginx
  NAME                      CREATED AT
  ingress-nginx             2023-08-15T10:25:53Z
  ingress-nginx-admission   2023-08-15T10:25:53Z
```

```yaml
# kubectl get role ingress-nginx --namespace ingress-nginx -o yaml
apiVersion: v1
items:
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.1.2
        helm.sh/chart: ingress-nginx-4.0.18
      name: ingress-nginx
      namespace: ingress-nginx
    rules:
      - apiGroups:
          - ""
        resources:
          - namespaces
        verbs:
          - get
      - apiGroups:
          - ""
        resources:
          - configmaps
          - pods
          - secrets
          - endpoints
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ""
        resources:
          - services
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingresses
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingresses/status
        verbs:
          - update
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingressclasses
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ""
        resourceNames:
          - ingress-controller-leader
        resources:
          - configmaps
        verbs:
          - get
          - update
      - apiGroups:
          - ""
        resources:
          - configmaps
        verbs:
          - create
      - apiGroups:
          - ""
        resources:
          - events
        verbs:
          - create
          - patch
```

```yaml
# kubectl get role ingress-nginx-admission --namespace ingress-nginx -o yaml
- apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    annotations:
      helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
      helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
    labels:
      app.kubernetes.io/component: admission-webhook
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
      app.kubernetes.io/version: 1.1.2
      helm.sh/chart: ingress-nginx-4.0.18
    name: ingress-nginx-admission
    namespace: ingress-nginx
  rules:
    - apiGroups:
        - ""
      resources:
        - secrets
      verbs:
        - get
        - create
```

### ServiceAccount RoleBindings

```yaml
# for ingress-nginx
apiVersion: v1
items:
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.1.2
        helm.sh/chart: ingress-nginx-4.0.18
      name: ingress-nginx
      namespace: ingress-nginx
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: ingress-nginx
    subjects:
      - kind: ServiceAccount
        name: ingress-nginx
        namespace: ingress-nginx
```

```yaml
# for ingress-nginx-admission
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    annotations:
      helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
      helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
    labels:
      app.kubernetes.io/component: admission-webhook
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
      app.kubernetes.io/version: 1.1.2
      helm.sh/chart: ingress-nginx-4.0.18
    name: ingress-nginx-admission
    namespace: ingress-nginx
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: ingress-nginx-admission
  subjects:
    - kind: ServiceAccount
      name: ingress-nginx-admission
      namespace: ingress-nginx
```

### ServiceAccount ClusterRoleBindings

```yaml
# for ingress-nginx
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
      - namespaces
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - networking.k8s.io
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - networking.k8s.io
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - networking.k8s.io
    resources:
      - ingressclasses
    verbs:
      - get
      - list
      - watch
```

```yaml
# for ingress-nginx-admission
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
rules:
  - apiGroups:
      - admissionregistration.k8s.io
    resources:
      - validatingwebhookconfigurations
    verbs:
      - get
      - update
```

## Ingress Controller Deployment

### Controller Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller

spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress

    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.1
      args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration

      # nginx needs these to read config data from within the pod
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace

      ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

## Ingress Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress

spec:
  type: NodePort
  selector:
    name: nginx-ingress
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
```
