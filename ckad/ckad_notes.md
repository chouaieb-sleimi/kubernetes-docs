# Certified Kubernetes Application Developer - CKAD

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Certified Kubernetes Application Developer - CKAD](#certified-kubernetes-application-developer---ckad)
  - [Section 1: Overview](#section-1-overview)
    - [Resources](#resources)
  - [Section 2: Core Concepts](#section-2-core-concepts)
    - [Docker vs ContainerD](#docker-vs-containerd)
    - [Containerd CLIs:](#containerd-clis)
    - [Namespaces](#namespaces)
  - [Section 3: Configuration](#section-3-configuration)
    - [Define, Build, Modify Container Images](#define-build-modify-container-images)
    - [Commands And Arguments](#commands-and-arguments)
    - [ConfigMap](#configmap)
      - [Intro: Environment Variables](#intro-environment-variables)
      - [Create ConfigMap](#create-configmap)
      - [Use ConfigMap](#use-configmap)
    - [Secrets](#secrets)
      - [Create Secrets](#create-secrets)
      - [Use Secret](#use-secret)
    - [Security](#security)
      - [Container Security](#container-security)
      - [SecurityContext](#securitycontext)
      - [Authentication](#authentication)
        - [Static Password File](#static-password-file)
        - [Static Token File](#static-token-file)
        - [Certificates](#certificates)
        - [KubeConfig](#kubeconfig)
      - [Authorization](#authorization)
        - [RBAC](#rbac)
          - [Roles and RoleBindings](#roles-and-rolebindings)
          - [ClusterRoles and ClusterRoleBindings](#clusterroles-and-clusterrolebindings)
        - [AdmissionController](#admissioncontroller)
          - [Dynamic AdmissionController](#dynamic-admissioncontroller)
    - [ServiceAccount](#serviceaccount)
      - [Create ServiceAcounts and Secrets](#create-serviceacounts-and-secrets)
      - [Use ServiceAccounts](#use-serviceaccounts)
    - [Resource Requirements](#resource-requirements)
      - [Limits and Requests](#limits-and-requests)
      - [LimitRanges](#limitranges)
    - [ResourceQuota](#resourcequota)
    - [Taints and Tolerations](#taints-and-tolerations)
      - [Taints (Node)](#taints-node)
      - [Tolerations](#tolerations)
    - [Node Selectors and Affinity](#node-selectors-and-affinity)
      - [Node Selectors](#node-selectors)
      - [Node Affinity](#node-affinity)
  - [Section 4: Multi-Container Pods](#section-4-multi-container-pods)
    - [Init Containers](#init-containers)
  - [Section 5: Observability And API Maintenance](#section-5-observability-and-api-maintenance)
    - [Observability](#observability)
      - [Readiness and Liveness Probes](#readiness-and-liveness-probes)
        - [Pod Status](#pod-status)
        - [Pod Conditions](#pod-conditions)
      - [Readiness Probe](#readiness-probe)
      - [Liveness Probe](#liveness-probe)
      - [Container Logging](#container-logging)
      - [Monitoring Cluster](#monitoring-cluster)
        - [Metrics server Overview](#metrics-server-overview)
        - [Metrics Server Deployment](#metrics-server-deployment)
  - [Section 6: Pod Design](#section-6-pod-design)
    - [Labels Selectors and Annotations](#labels-selectors-and-annotations)
    - [Rolling Updates and Rollbacks in Deployments](#rolling-updates-and-rollbacks-in-deployments)
    - [Jobs and CronJobs](#jobs-and-cronjobs)
      - [Jobs](#jobs)
      - [CronJobs](#cronjobs)
  - [Section 7: Services and Networking](#section-7-services-and-networking)
    - [Ingress](#ingress)
      - [Ingress Controller](#ingress-controller)
        - [Ingress ConfigMap](#ingress-configmap)
        - [Ingress ServiceAccount](#ingress-serviceaccount)
        - [Ingress Controller Deployment](#ingress-controller-deployment)
        - [Ingress Service](#ingress-service)
      - [Ingress Resources](#ingress-resources)
        - [Single URL - Single paths - Single backend](#single-url---single-paths---single-backend)
        - [Single URL - Mutliple paths - Mutliple backends](#single-url---mutliple-paths---mutliple-backends)
        - [Multiple URLs - Mutliple backends](#multiple-urls---mutliple-backends)
    - [NetworkPolicy](#networkpolicy)
    - [Port Forwarding](#port-forwarding)
  - [Section 8: State Persistance](#section-8-state-persistance)
    - [Volume](#volume)
    - [PersistentVolume](#persistentvolume)
    - [PersistentVolumeClaim](#persistentvolumeclaim)
    - [StorageClass](#storageclass)
    - [StatefulSets](#statefulsets)
    - [Headless Services](#headless-services)
    - [volumeClaimTemplates](#volumeclaimtemplates)
  - [Section 9: Post Sep-2021 Changes](#section-9-post-sep-2021-changes)
    - [Operator Framework](#operator-framework)
      - [API Maintenance](#api-maintenance)
        - [APIs hierarchy](#apis-hierarchy)
        - [API Versions](#api-versions)
        - [API Deprecation](#api-deprecation)
      - [CustomResourceDefinition](#customresourcedefinition)
      - [CustomControllers](#customcontrollers)
      - [Operators](#operators)
    - [Deployment Strategies](#deployment-strategies)
      - [Blue Green Deployments](#blue-green-deployments)
      - [Canary Deployments](#canary-deployments)
    - [Helm](#helm)
  - [Section 10: Labs](#section-10-labs)

<!-- /code_chunk_output -->

## Section 1: Overview

### Resources

- Certified Kubernetes Application Developer: https://www.cncf.io/certification/ckad/

- Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

- Exam Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

Keep the code - 20KLOUD handy while registering for the CKA or CKAD exams at Linux Foundation to get a 20% discount.

- Kubernetes The Hard Way
  https://github.com/mmumshad/kubernetes-the-hard-way

## Section 2: Core Concepts

### Docker vs ContainerD

![docker vs. podman](img/section-02_docker-vs-podman.jpeg)
K8s **supports containerd** and **not docker**.

Supported K8s runtimes:

- containerd
- CRI-O
- Docker Engine (cri-dockerd)

### Containerd CLIs:

- **ctr**
  **purpose: debugging**
  **community: containerd**
  **works with: containerd**

  - comes w/ containerd
  - not user friendly
  - limited features

- **nerdctl**
  **purpose: general purrpose**
  **community: containerd**
  **works with: containerd**
  - docker-like cli
  - supports docker-compose
  - supports containerd features:
    - encrypted container images
    - lazy pulling
    - image signing and verifying
    - namespaces w/ k8s

**CRI CLIs:**

- **crictl**
  **purpose: debugging**
  **community: kubernetes**
  **works with: CRI compatible runtimes**
  - installed separately
  - inspect and debug runtimes
    - not to create containers
      (any created containers will be removed by kubelet)
  - cross containre runtimes
  - should manually set runtime endpoints (`crictl --runtime-endpoint`):
    unix:///run/containerd/containerd.sock
    unix:///run/crio/crio.sock
    unix:///var/run/cri-dockerd.sock

### Namespaces

Provides a mechanism for isolating groups of resources within a single cluster.

- They need to be unique within a namespace, but not across namespaces.
- Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

**Cross-namespace object name format:**
format: `<object-name>.<namespace-name>.<object-type>.<cluster-domain>`
example: `db-service.dev.service.cluster.local`

## Section 3: Configuration

### Define, Build, Modify Container Images

### Commands And Arguments

`IMAGE/entrypoint = K8S/command` and `IMAGE/cmd = K8S/args`

Containerfile

    ENTRYPOINT ["python", "app.py"]
    CMD ["--color", "red"]

equivalent in **Pod YAML**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["python", "app.py"]
        args: ["--color", "red"]

### ConfigMap

allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

consumeed as:

- environment variables,
- command-line arguments,
- as configuration files in a volume.

#### Intro: Environment Variables

example

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name: simple-webapp-color
          image: simple-webapp-color
          ports:
            - containerPort: 8080
          env:
            - name: APP_COLOR
              value: red

**ENV Value Types:**

- **Plain Key Value pair:**

```
env:
  - name: APP_COLOR
    value: red
```

- **ConfigMap:**

```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        ...
```

- **Secrets:**

```
env:
  - name: APP_COLOR
    secretKeyRef:
      ...
```

#### Create ConfigMap

- imperative approach

      # from-literal
      kubectl create configmap <config-name> \
        --from-literal=APP_COLOR=red \
        --from-literal=APP_MOD=prod

      # from file
      kubectl create configmap <config-name> \
        --from-file=app_config.properties

- **declarative approach**
  config-map_definition.yaml

      apiVersion: v1
      kind: Pod
      metadata:
        name: app-config-map
      data:
        APP_COLOR: red
        APP_MOD: prod

#### Use ConfigMap

inject configmap

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          envFrom:
            - configMapRef:
                name: app-config-map

inject single variable

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          env:
            - name: APP_COLOR
              valueFrom:
                configMapKeyRef:
                  name: app-config-map
                  key: APP_COLOR

inject configmap from volume

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          volumes:
            - name: app-config-volume
              configMap:
                name: app-config-map

### Secrets

- not encrypted, only encoded
- secrets are not encrypted in ETCD
  - configure encryption at rest (they are stored encrypted in ETCD)
    see: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
- anyone that cat create pods/deployments can see secrets
  - configure least-privilege access to secrets - RBAC
- consider third-party secrets store providers (AWS, Azure, GCP, Vault, etc.)

#### Create Secrets

    # encode data
    echo -n 'mypassword' | base64

    # decode data
    echo -n 'mypassword' | base64 --decode

- imperative approach

      # from-literal
      kubectl create secret generic <secret-name> \
        --from-literal=DB_Host=mysql \
        --from-literal=DB_User=root \
        --from-literal=DB_Pwd=password


      # from file
      kubectl create secret generic <secret-name> \
        --from-file=app_secrets.properties

- **declarative approach**
  config-map_definition.yaml

      apiVersion: v1
      kind: Pod
      metadata:
        name: app-secrets
      data:
        DB_Host: bXlzcWw=
        DB_User: cm9vdA==
        DB_Pwd: cGFzc3dvcmQ=

#### Use Secret

inject secret

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          envFrom:
            - secretRef:
                name: app-secrets

inject single variable

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          env:
            - name: APP_COLOR
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: DB_Host

inject secret from volume

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
          ...
          volumes:
            - name: app-secrets-volume
              secret:
                secretName: app-secrets

### Security

Secure:

- **hosts**

  - ssh root access disabled
  - ssh password access disabled
  - ssh key based auth

- **cluster:**

  - Set up certs for inter-cluster-components TLS communication
  - Secure inter-pod comm using NetworkPolicies

**Authentication:** (who can access)

- Files - Username and password
- Files - Username and tokens
- Certificates
- Axternal auth providers - LDAP
- Service accounts (machines)

**Authorization:** (what can they do)

- RBAC (Role Based Access Control)
- ABAC (Atribute Based Access Control)
- Node Auth
- Webhook mode

#### Container Security

User capabilities list

    /usr/include/linux/capability.h

Override user privileges in podman run cmd

    # add privilege flag
    podman run --cap-add MAC_ADMIN ubuntu

    # drop privilege flag
    podman run --cap-drop KILL ubuntu

    # add ALL privileges
    podman run --privileged ubuntu

#### SecurityContext

- `runAsUser`
- `capabilities`
  **container level**
  - `add`
  - `drop`

**pod level** security context.

- _applies to all containers_
- _capabilities **only available in container level**_

      apiVersion: v1
      kind: Pod
      metadata:
        ...
      spec:
        securityContext:
          runAsUser: 1012
        containers:
          - name:
            ...

**container level** security context. _if pod security context exist, the container context is applied_

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name:
          ...
          securityContext:
            runAsUser: 1000
            capabilities:
              add: ["MAC_ADMIN"]
              drop:
                - KILL

#### Authentication

Types of accounts:

- Service ccounts (bots and machines)
- User accounts
  **not managed by k8s**, it is **handled externally** via:
  - static password file\* (Deprecated in 1.19)
  - static token file\* (Deprecated in 1.19)
  - certificates
  - identity service (ex: LDAP)

> Notes:
>
> - file auth not recommended
> - consider using volume mount while providing the auth file in a kubeadm setup
> - setup RBAC for new users

##### Static Password File

static password file sample

    cat user-details.csv

    # password,usernae,uid,group (optional)
    password123,user1,u0001,group1
    password123,user2,u0002,group2
    password123,user3,u0003,group3
    password123,user4,u0004,group4

use static password file in `kube-apiserver`

    /usr/local/bin/kube-apiserver \
      ...
      --base-file-auth=user-details.csv
      ...

use account auth in API call

    curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"

##### Static Token File

static token file sample

    cat user-token-details.csv

    # password,usernae,uid,group (optional)
    b026324c6904b2a9cb4b88d6d61c81d1,user1,u0001,group1
    26ab0db90d72e28ad0ba1e22ee510510,user2,u0002,group2
    6d7fce9fee471194aa8b5b6e47267f03,user3,u0003,group3
    48a24b70a0b376535542b996af517398,user4,u0004,group4

use static token file in `kube-apiserver`

    /usr/local/bin/kube-apiserver \
      ...
      --token-auth-file=user-token-details.csv
      ...

use account auth in API call

    curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer b026324c6904b2a9cb4b88d6d61c81d1"

##### Certificates

use cert auth in API call

    curl https://my-kube-playground:6443/api/v1/pods \
    --key admin.key \
    --cert admin.crt \
    --cacert ca.crt

use cert auth in kubectl

    kubectl get pods \
      --server my-kube-playground:6443 \
      --client-key admin.key \
      --client-certificate admin.crt \
      --certificate-authority ca.crt

##### KubeConfig

**path:** `$HOME/.kube/config`
**sections:**

- **Clusters**
  example: `Development`,`Production`,`Google`, `KubePlayground`
- **Contexts**
  example: `Admin@Prodction`,`Dev-user@Google`, `KubeAdmin@KubePlayground`
- **Users**
  example: `Admin`,`Dev-user`,`Prod-user`, `KubeAdmin`

.`kube/config` sample 1

    apiVersion: v1
    kind: Config
    current-context: dev-user@google # kubectl default context

    clusters:
    - development
    - production
    - google
    - kubeplayground

    contexts:
    - admin@prodction
    - dev-user@google
    - kubeadmin@kubeplayground

    users:
    - admin
    - dev-user
    - prod-user
    - kubeadmin

`.kube/config` sample 2

    # <cert-base64-data>: cat ca.crt | base64
    apiVersion: v1
    kind: Config
    current-context: admin@production # kubectl default context

    clusters:
    - production
      cluster:
        certificate-authority: /etc/kubernetes/pki/ca.crt
        # OR
        certificate-authority-data: <cert-base64-data>
        server: https://172.17.0.51:6443

    contexts:
    - admin@prodction
      context:
        cluster: production
        user: admin
        context: finance

    users:
    - admin
      user
        client-certificate: /etc/kubernetes/pki/users/admin.crt
        client-key: /etc/kubernetes/pki/users/admin.key

`.kube/config` sample 3

    apiVersion: v1
    kind: Config
    preferences: {}
    current-context: kubernetes-admin@kubernetes
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJT.....tCg==
        server: https://controlplane:6443
      name: kubernetes
    contexts:
    - context:
        cluster: kubernetes
        user: kubernetes-admin
      name: kubernetes-admin@kubernetes
    users:
    - name: kubernetes-admin
      user:
        client-certificate-data: LS0tLS1CRUdJTiBDR.....S0tCg==
        client-key-data: LS0tLS1CRUdJ.....ktLS0tLQo=

#### Authorization

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

##### RBAC

Steps:

- create `Role`/`ClusterRole`
- create `RoleBinding`/`ClusterRoleBinding`

get user access

    kubectl auth can-i [--as dev-user] create deployments
    kubectl auth can-i [--as dev-user] delete pods

###### Roles and RoleBindings

specific for namespaced-scoped resources:

- pods
- replicasets
- jobs
- deployments
- services
- secrets
- roles
- rolebindings
- configmaps
- PVC

creates role that can view, create, delete pods and create ConfigMaps

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: developer-role
    rules:
      - apiGroups: [""] # "" indicates the core API group
        resources: ["pods"]
        verbs: ["list, "get", "create", "update", "watch", "delete"]
        resourceName: ["front-end", "back-end"]
      - apiGroups: [""]
        resources: ["ConfigMaps"]
        verbs: ["create"]

associate role with user through a RoleBinding

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      # limits user access to this namespace
      namespace: default
      name: dev-user-binding
    subjects:
      - kind: User
        name: dev-user # "name" is case sensitive
        apiGroup: rbac.authorization.k8s.io
    roleRef:
      # "roleRef" specifies the binding to a Role / ClusterRole
      kind: Role #this must be Role or ClusterRole
      name: developer-role # this must match the name of the Role or ClusterRole you wish to bind to
      apiGroup: rbac.authorization.k8s.io

###### ClusterRoles and ClusterRoleBindings

Roles for cluster-scoped resources\*:

- nodes
- PV
- clusterroles
- clusterrolebindings
- namespaces

_\*: ClusterRoles can be created for namespaced-resources, the role will apply across ALL namespaces for that resource_

create cluster role

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      # "namespace" omitted since ClusterRoles are not namespaced
      name: cluster-admin
    rules:
      - apiGroups: [""]
        resources: ["nodes"]
        verbs: ["get", "watch", "list", "create", "delete"]

associate role with user using ClusterRoleBinding

    apiVersion: rbac.authorization.k8s.io/v1
    # This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
    kind: ClusterRoleBinding
    metadata:
      name: read-secrets-global
    subjects:
      - kind: Group
        name: manager # Name is case sensitive
        apiGroup: rbac.authorization.k8s.io
      - kind: User
        name: cluster-admin
        apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: rbac.authorization.k8s.io

##### AdmissionController

intercepts requests to the kubernetes api server

- prior to persistence of the object
- after the request is authenticated and authorized.
- admission controllers may be **validating**, **mutating**, **or both**:
  - **mutating controllers** may modify request objects
  - **validating controllers** may not

![Admission Controller Phases](./img/admission-controller-phases.png)

**Pre-built** admission controllers:

- AlwaysPullImages
- DefaultStorageClass
- EventRateLimit
- NamespaceExists
- NamespaceAutoProvision
- DefaultStorageClass
- ...

view enabled admission controllers

    kube-apiserver -h | grep enable-admission-plugin

    # in a kubeadm setup, run in kube apiserver controlplane pod
    kubectl exec kube-apiserver-controlplane -n kube-system -- \
      kube-apiserver -h | grep enable-admission-plugin

add admission controller via command

    /usr/local/bin/kube-apiserver \
      ...
      --enable-admission-plugins=NodeRestriction,...
      --disable-admission-plugins=DefaultStorageClass,...
      ...

add admission controller via kube-apiserver definition file

    apiVersion: v1
    kind: Pod
    metadata:
      name: kube-apiserver
      namespace: kube-system

    spec:
      containers:
      - command:
        - kube-apiserver
        - ...
        - --enable-admission-plugins=NodeRestriction,...
        - --disable-admission-plugins=DefaultStorageClass,...
        - ...
        image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
        name: kube-apiserver
        ...

###### Dynamic AdmissionController

In addition to

- compiled-in admission controller plugins
- developed-as-extensions admission controller; run as webhooks configured at runtime

**Admission webhooks**
HTTP callbacks that receive admission requests and do something with them.
You can define two types of admission webhooks:

- **validating admission webhook**
  modify objects sent to the API server to enforce custom defaults
- **mutating admission webhook**
  can reject requests to enforce custom policies

**Control order:**
`mutating admission webhooks` -> `kube-apiserver validation` -> `validating admission webhooks`

**Dynamic AdmissionController deployment**

1.  deploy admission webhook server

_sample admission server_
https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go

sample deployment

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webhook-server
      namespace: webhook-demo
      labels:
        app: webhook-server
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: webhook-server
      template:
        metadata:
          labels:
            app: webhook-server
        spec:
          securityContext:
            runAsNonRoot: true
            runAsUser: 1234
          containers:
          - name: server
            image: stackrox/admission-controller-webhook-demo:latest
            imagePullPolicy: Always
            ports:
            - containerPort: 8443
              name: webhook-api
            volumeMounts:
            - name: webhook-tls-certs
              mountPath: /run/secrets/tls
              readOnly: true
          volumes:
          - name: webhook-tls-certs
            secret:
              secretName: webhook-server-tls

Note: this webhook deployment:

- Denies all request for pod to run as root in container if no securityContext is provided.
- If no value is set for runAsNonRoot, a default of true is applied, and the user ID defaults to 1234
- Allow to run containers as root if runAsNonRoot set explicitly to false in the securityContext

2.  create webhook service

sample service

    apiVersion: v1
    kind: Service
    metadata:
      name: webhook-server
      namespace: webhook-demo
    spec:
      selector:
        app: webhook-server
      ports:
        - port: 443
          targetPort: webhook-api

3.  configure webhook on k8s

validating webhook configuration sample

    apiVersion: admissionregistration.k8s.io/v1
    kind: ValidatingWebhookConfiguration
    metadata:
      name: "pod-policy.example.com"
    webhooks:
    - name: "pod-policy.example.com"
      clientConfig:
        # if webhook server deployed outside the cluster
        #url: <webhook-server-url>
        service:
          namespace: "webhook-namespace"
          name: "webhook-service"
        caBundle: <CA_BUNDLE>       # to comm w/ the webhook server
      rules:
      - apiGroups:   [""]
        apiVersions: ["v1"]
        operations:  ["CREATE"]
        resources:   ["pods"]
        scope:       "Namespaced"
      admissionReviewVersions: ["v1"]
      sideEffects: None
      timeoutSeconds: 5

sample mutating webhook configuration sample

    apiVersion: admissionregistration.k8s.io/v1
    kind: MutatingWebhookConfiguration
    metadata:
      name: demo-webhook
    webhooks:
      - name: webhook-server.webhook-demo.svc
        clientConfig:
          service:
            name: webhook-server
            namespace: webhook-demo
            path: "/mutate"
          caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQekNDQWllZ0F3SUJBZ0lVRThTdVA5OEpKWlREcFp1Sm9NZHdGZG9JcDlFd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0x6RXRNQ3NHQTFVRUF3d2tRV1J0YVhOemFXOXVJRU52Ym5SeWIyeHNaWElnVjJWaWFHOXZheUJFWlcxdgpJRU5CTUI0WERUSXpNRGd5T1RFd05UazBNVm9YRFRJek1Ea3lPREV3TlRrME1Wb3dMekV0TUNzR0ExVUVBd3drClFXUnRhWE56YVc5dUlFTnZiblJ5YjJ4c1pYSWdWMlZpYUc5dmF5QkVaVzF2SUVOQk1JSUJJakFOQmdrcWhraUcKOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXhtVU13a2NBRHJFdURkQnVWSk9kbDJUaDFqWittS0tzc0NVQgpmbFR4b0pBeDZENHNRalJpSUh1R3R1cEFCNG9YS29pTHhTQ0VMNVEvenJzMzd5aVNpellwRlA1YnI0NElpejNQCjhUNGViemJzam9ac2YxcmpiNFRKSWZVUWd6YlJYRnIwOCs1VmNnREEwTzdla280R0dNSGw2WnFkTkVRUXJPTzMKbFlxUUlTenBlUm43eGowbXJ0Wk8raXBMeHIxdXJXcWoyTG5nd3RqcmVkaFFrRGVKNjh6dGdXaFlkRXRNOHhzeQovSWYvUGV6YU8yUW1CRldmaW9CMjIzQWswc2ZjTDdIY09DcWRyZ3B4b2xva1VVMGkxUG5wRzFVTmI0eEhPNW91Cksrb3NCalFXVTlDT1R0dnZpem5RL2grYnlFZ01IM3BTa2pKUGI2ZnBlWDJST3BkaXpRSURBUUFCbzFNd1VUQWQKQmdOVkhRNEVGZ1FVeFJXK2NOSHNhRVJEUi9pTWhUSDFuRlFIWUIwd0h3WURWUjBqQkJnd0ZvQVV4UlcrY05IcwphRVJEUi9pTWhUSDFuRlFIWUIwd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DCkFRRUFlN29WZlJCS1FRTW1XTy9VMG8vWGZqTG5BcForWkhKZG1BaGR4VTMrVVRPZFQzK1Jpa0JESW5DeFFPVGEKek8yYnJRcjZ4MllyQkpDNGFNNDRGb3hjd0RJSkxuak1WNlJYc2JuMHdDU0V3MGxYa0lpWVl4ZGkzbFEzWGVlLwo1TGRVa004TEZnOGFTMjc2Q0Q1R2M3RUdSRXhSWWVXUmprQWg1TGgrMnowL2p1Q3RmTUN0Tjk4YkphYUNYcFdlClVldUJDeldGUXlPRWJoK2Uxa1RDMTlGSnFzRTdrZWtBbGhZRzZieTAzWU9kR0NLM3ZwTVdJK3UzM3RzcU9xajgKbWR5aVpLemlHRlJmNXN3ZGFPY1pEZDllOUNmaVhVeHpJdmFFZUpQR1FiUlRMc3ltOCs4K0dkRlhBQVIrbFZyZQo1LythSlloNnpscm05NnhLLzhGSFpxYkc5UT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
        rules:
          - operations: [ "CREATE" ]
            apiGroups: [""]
            apiVersions: ["v1"]
            resources: ["pods"]
        admissionReviewVersions: ["v1beta1"]
        sideEffects: None

pod samples (to test mutating admission controller above)

    # A pod with no securityContext specified.
    # Without the webhook, it would run as user root (0). The webhook mutates it
    # to run as the non-root user with uid 1234.
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-with-defaults
      labels:
        app: pod-with-defaults
    spec:
      restartPolicy: OnFailure
      containers:
        - name: busybox
          image: busybox
          command: ["sh", "-c", "echo I am running as user $(id -u)"]

    ---
    # A pod with a securityContext explicitly allowing it to run as root.
    # The effect of deploying this with and without the webhook is the same. The
    # explicit setting however prevents the webhook from applying more secure
    # defaults.
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-with-override
      labels:
        app: pod-with-override
    spec:
      restartPolicy: OnFailure
      securityContext:
        runAsNonRoot: false
      containers:
        - name: busybox
          image: busybox
          command: ["sh", "-c", "echo I am running as user $(id -u)"]

    ---
    # A pod with a conflicting securityContext setting: it has to run as a non-root
    # user, but we explicitly request a user id of 0 (root).
    # Without the webhook, the pod could be created, but would be unable to launch
    # due to an unenforceable security context leading to it being stuck in a
    # 'CreateContainerConfigError' status. With the webhook, the creation of
    # the pod is outright rejected.
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-with-conflict
      labels:
        app: pod-with-conflict
    spec:
      restartPolicy: OnFailure
      securityContext:
        runAsNonRoot: true
        runAsUser: 0
      containers:
        - name: busybox
          image: busybox
          command: ["sh", "-c", "echo I am running as user $(id -u)"]test admission controller pod

    kubectl logs <pod-name>
    kubectl get po <pod-name> -o yaml | grep -iA3 securitycontext

    kubectl create -f /root/pod-with-conflict.yaml
      Error from server: error when creating "/root/pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)

### ServiceAccount

generates an access token in a secret object upon creation.

**serviceAccount workflow:**

- create serviceAccount
- generate access token secret (if necessary)
- mount token as volume to pods.

**post v1.22:** serviceAccounts no longer rely on secret tokens. tokens generated by the **TokenRequestAPI** are:

- audience bound
- time bound
- object bound

**pre v1.24:** [**serviceAccount** [**secret** [**token**(doesn't expire)]]]
**post v1.24:** [**serviceAccount** [**token**(default expire after 1h)]]

**Documentation**:

- Service account token Secrets:
  https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets
- Managing Service Accounts:
  https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

#### Create ServiceAcounts and Secrets

create an access token in a secret object **post v1.24**:
_serviceAccount must be created first_

    # expires after 1 hour
    kubectl create token jenkins-sa

    # doesn't expire
    apiVersion: v1
    kind: Secret
    type: kubernetes.io/service-account-token
    metadata:
      name: jenkins-sa
      annotations:
        kubernetes.io/service-account.name: jenkins-sa

see sa token

    kubectl describe serviceaccount jenkins-sa | grep -i token

    jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64 | fromjson' <<< <secret_token>

#### Use ServiceAccounts

use sa token

    curl htts://192.168.56.70:6443/api -insecure \
    --header "Authorization: Bearer <sa_access-token>"

use sa in a pod

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name:
          ...
      serviceAccountName: jenkins-sa

disable default sa automount

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name:
          ...
          automountServiceAccountToken: false

### Resource Requirements

#### Limits and Requests

When exceeding **CPU**, pods are throttled.
When exceeding **MEM**, pods are terminated to free memory and are re-created because of an **OOM**.

**CPU** must be >0.1**cpu** or >1**m** (`1cpu = 1000m`; `m: milli`)
**MEM** can be 256Mi = 268 M = 268435456

possible **requests/limits scenarios:**

- **NO REQUESTS / NO LIMITS**

  - **CPU**
    pods can consume all the resources and starve others
  - **MEM**
    pods can consume all the resources and starve others

- **NO REQUESTS / LIMITS**

  - **CPU**
    requests = limits
  - **MEM**
    requests = limits

- **REQUESTS / LIMITS**

  - **CPU**
    requests are guaranteed, exceeding limits results in **throtelling**
  - **MEM**
    requests are guaranteed, exceeding limits results in **recreation**

- **REQUESTS / NO LIMITS**

  - **CPU**
    exceeding limits is permitted when it doesn't starve others of requests
  - **MEM**
    exceeding limits is permitted when it doesn't starve others of requests

set container **resource request** and **resource limits**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name: ...
          image: ...
          ports:
            - containerPort: ...
          resources:
            requests:
              memory: "4Gi"
              cpu: 2
            limits:
              memory: "6Gi"
              cpu: 3

#### LimitRanges

- namespaced
- default limit values for pods created without requests or limits.
- affect **only newly created pods.**

create cpu LimitRange

    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-resource-constraint
    spec:
      limits:
      - default:          # default limit
          cpu: 500m
        defaultRequest:   # default request
          cpu: 500m
        max:              # max limit that can be set on a container
          cpu: 1
        min:              # min request a container can make
          cpu: 100m
        type: Container

create memory LimitRange

    apiVersion: v1
    kind: LimitRange
    metadata:
      name: memory-resource-constraint
    spec:
      limits:
      - default:          ## default limit
          memory: 1Gi
        defaultRequest:   # default request
          memory: 1Gi
        max:              # max limit that can be set on a container
          memory: 1Gi
        min:              # min request a container can make
          memory: 500Mi
        type: Container

### ResourceQuota

- namespaced
- limits total resource usage on the namespace

create ResourceQuota

    apiVersion: v1
    kind: LimitRange
    metadata:
      name: my-resource-quota
    spec:
      hard:
        requests.cpu: 4
        requests.memory: 4Gi
        limits.cpu: 10
        limits.memory: 10Gi

### Taints and Tolerations

Are used to set **restrictions on what pods nodes CAN accept (NOT MUST)**.

#### Taints (Node)

see node's taints

    kubectl describe nodes <node-name> | grep -i taint

`taint-effect` is what happends to PODs **that DO NOT TOLERATE this taint**:

- **NoSchedule**
  pods will not be scheduled on the node
- **PreferNoSchedule**
  try to avoid placing pod on node
- **NoExecute**
  new pods will not be scheduled on the node, existing pods that don't tolerate the taint are evicted

  **create a taint** on a node

  kubcetl taint nodes <node-name> key=value:<taint-effect>

  kubcetl taint nodes node01 app=blue:NoSchedule

#### Tolerations

add toleration to a pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
    spec:
      containers:
        - name: nginx-container
          image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"

### Node Selectors and Affinity

Restrict to run on particular node(s), or to prefer to run on particular nodes.

#### Node Selectors

with `nodeSelector`, you can add the nodeSelector field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

create **node label to be used as a selector** for pods

    kubectl label nodes <node-name> <label-key>=<label-value>

run pod on selected nodes

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
    spec:
      containers:
        - name: data-processor
          image: data-processor
      nodeSelector:
        <label-key>: <label-value>
        size: Large

#### Node Affinity

conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

- **requiredDuringSchedulingIgnoredDuringExecution**:
  The scheduler can't schedule the Pod unless the rule is met. This **functions like nodeSelector, but with a more expressive syntax.**

- **preferredDuringSchedulingIgnoredDuringExecution**:
  The scheduler tries to find a node that meets the rule. **If a matching node is not available, the scheduler still schedules the Pod.**

Possible node affinities:

    DuringScheduling      DuringExecution
    -----------------------------------------
    Required              Ignored
    Preferred             Ignored
    Required              Required

add node affinity to a pod

      apiVersion: v1
      kind: Pod
      metadata:
        name: myapp-pod
      spec:
        containers:
          - name: data-processor
            image: data-processor
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                # SPECIFY POSSIBLE labels
                - key: size
                  operator: In
                  values:
                  - Large
                  - Medium

                # NOT IN operator
                - key: size
                  operator: NotIn
                  values:
                  - Small

                # IF Small nodes don't have labels
                - key: size
                  operator: Exists

example:

    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: blue
    spec:
      replicas: 3
      selector:
        matchLabels:
          run: nginx
      template:
        metadata:
          labels:
            run: nginx
        spec:
          containers:
          - image: nginx
            imagePullPolicy: Always
            name: nginx
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                  - key: color
                    operator: In
                    values:
                    - blue

## Section 4: Multi-Container Pods

- for tightly coupled containers that need to share resources.
- containers share
  - lifecycle,
  - network space (they can reference each other with `localhost`)
  - storage volumes.

Multi-Container Pods Design Patterns

- **SideCar**
  The sidecar pattern consists of a main application + a helper container with a responsibility that is essential to your application, but **is not necessarily part of the application itself.**

- **Adapter**
  The adapter pattern is used to **standardize and normalize application output or monitoring data for aggregation.**

- **Ambassador**
  The ambassador pattern is a useful way to **connect containers with the outside world.**
  An ambassador container is essentially a proxy that allows other containers to connect to a port on localhost while the ambassador container can proxy these connections to different environments depending on the cluster's needs.

### Init Containers

Used to run a process that runs to completion in a container.

can be:

- a task that will be **run only one time when the pod is first created**
- a process that waits for an external service or database to be up before the actual application starts

**define an initContainer** in a pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:

      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']

      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']

- `initContainers` processs must run to a completion `containers` starts.
- multiple `initContainers` run one at a time in sequential order.

If one of the `initContainers` fail to complete, Pod restarts repeatedly until the `initContainers` succeeds.

`initContainers` example

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:

      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']

      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']

      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']

Read more about initContainers here.
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

## Section 5: Observability And API Maintenance

### Observability

#### Readiness and Liveness Probes

Failing:

- **liveness probe:** restart container
- **readiness probe:** stop container from serving traffic

##### Pod Status

**Pod States:**

- Pending (Pod not scheduled)
- ContainerCreating
- Running

get pod status

    kubectl get pods
    kubectl describe pod <pod-name> | grep -i status

##### Pod Conditions

Pod conditions compliment pod status. **true or false.**

**Pod conditions:**

- PodScheduled
- Initialized
- ContainerReady (containers are running)
- Ready (pod is running)

get pod conditions

    kubectl describe pod <pod-name> | grep -iA5 conditions

#### Readiness Probe

**when is a container ready to accept traffic?**
**example:** Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

probe types:

- **HTTP request test**
- **Port test**
- **Script execution**

**readiness probe** sample

    apiVersion: v1
    kind: Pod
    metadata:
      name: simple-webapp
      labels:
        app: simple-webapp

    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
        ports:
        - containerPort: 8080

        # HTTP Test
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 8080
          initialDelaySeconds: 10   # wait before probes begin
          periodSeconds: 5          # deplay bettween probes
          failureThreshold: 8       # retries before failure declared; default is 3

        # TCP Test
        readinessProbe:
          tcpSocket:
            port: 3306

        # Script execution
        readinessProbe:
          exec:
            command:
            - cat
            - /app/is_ready

#### Liveness Probe

**when is the application healthy? / when to restart a container?**
**example:** catch deadlocks, where an application is running, but unable to make progress.

**Liveness probe types:**

- HTTP request test
- Port test
- Script execution

**liveness probe** sample

    apiVersion: v1
    kind: Pod
    metadata:
      name: simple-webapp
      labels:
        app: simple-webapp

    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
        ports:
        - containerPort: 8080

        # HTTP Test
        livenessProbe:
          httpGet:
            path: /api/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 8     # default is 3

        # TCP Test
        livenessProbe:
          tcpSocket:
            port: 3306

        # Script execution
        livenessProbe:
          exec:
            command:
            - cat
            - /app/is_ready

#### Container Logging

get a container's logs

    # <container-name> is necessary for multi-container pods.
    kubectl logs -f <pod-name> [<container-name>]

#### Monitoring Cluster

Monitoring metrics:

- **node-level metrics**

  - node number
  - node health
  - node performance metrics
    - cpu
    - memory
    - network
    - disk

- **pod-level metrics**
  - number of pods
  - pods performance metrics
    - cpu
    - memory

K8s **monitoring solutions:**

- heapster **(deprecated)**
- metric server
- prometheus
- ELK stack
- data dog (proprietary)
- dynatracee (proprietary)

##### Metrics server Overview

Is a slimmed down version of heapster. can only be 1 metrics server per k8s cluster. An **In-Memory monitoring solution**; doesn't store logs and mterics data on disk.

Uses a `kubelet` component `cAdvisor`; retrieves pod performance metrics and expose them through kubelet api to metrics server.

##### Metrics Server Deployment

deploy metrics server in minikube

    minikube addons eable metrics-server

deploy metrics server in other cluster

    # clone yaml deployment files
    # contain set of pods, services and roles
    git clone https://github.com/kubernetes-incubator/metrics-server.git

    # deploy metrics server
    kubectl create -f deploy/1.8+/

get performance metrics

    kubectl top node
    kubectl top pod

## Section 6: Pod Design

### Labels Selectors and Annotations

The API currently supports **two types of selectors: equality-based and set-based**.
A label selector can be made of multiple requirements which are comma-separated.

You can use Kubernetes **annotations** to attach **arbitrary non-identifying metadata** to objects.
Clients such as tools and libraries can retrieve this metadata.

### Rolling Updates and Rollbacks in Deployments

Rollout strategies:

- **Recreate** strategy
- **RollingUpdate** (default strategy)

apply rollout

    # apply yaml file
    kubectl apply -f deployment_def.yaml [--record]

    # edit deployment
    kubectl edit deploy <deployment-name> [--record]

    # via command (will not update file)
    kubectl set image deploy <deployment-name> [--record] \
      nginx-container=nginx:1.9.1

get **rollout status and history**

    # describe deployment
    kubectl describe deploy <deployment-name>

    # rollout status
    kubectl rollout status deploy <deployment-name> [--revision <revision-num>]

    # rollout history
    kubectl rollout history deploy <deployment-name> [--revision <revision-num>]

rollback latest revision

    # rollback update
    kubectl rollback deploy <deployment-name>

### Jobs and CronJobs

#### Jobs

- creates one or more Pods and will **retry execution until one or many successes.**
- **deleting** a Job will **clean up the Pods it created.**
- **suspending** a Job will **delete its active Pods** until the Job is resumed again.

define a pod **restart policy**

    apiVersion: v1
    kind: Pod
    metadata:
      name: math-pod

    spec:
      containers:
      - name: math-add
        image: ubuntu
        command: ['expr', '3', '+', '2']
      restartPolicy: Always # Always, Never, OnFailure

create job

    apiVersion: batch/v1
    kind: Job
    metadata:
      name: math-add-job

    spec:
      completions: 3    # retries until 3 successful completions
      parallelism: 3
      backoffLimit: 4   # default is 6
      template:
        spec:
          containers:
          - name: math-add
            image: ubuntu
            command: ['expr', '3', '+', '2']
          restartPolicy: Never

#### CronJobs

Creates Jobs on a repeating schedule.

create CronJob

    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: hello
    spec:
      schedule: "* * * * *"
      jobTemplate:
        spec:
          template:
            spec:
              containers:
              - name: hello
                image: busybox:1.28
                imagePullPolicy: IfNotPresent
                command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
              restartPolicy: OnFailure

## Section 7: Services and Networking

Main types of services:

- **NodePort**
- **ClusterIP**
- **LoadBalancer**

### Ingress

- exposes HTTP and HTTPS **routes from outside the cluster to services within the cluster.**
- traffic routing by rules in Ingress resource.
- provides:
  - load balancing
  - SSL termination
  - name-based virtual hosting

**Ingress components:**

- **Ingress controller** (Ingress runtimes)
  And ingress controller is a special LoadBalancer container deployment customized for Ingress. They can be:

  - Cloud LoadBalancers (supported)
  - Nginx (supported)
  - HAProxy
  - traefik
  - Istio
  - Contour

- **Ingress resources**
  Routing rules for the ingress controller

#### Ingress Controller

**Deployment steps:**

- Create ingress config map
- Prepare ingress service account (for monitoring ingress resource changes)
- Create controller deployment
- Create ingress service

##### Ingress ConfigMap

create ingress config map

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nginx-configuration

##### Ingress ServiceAccount

**prepare ingress service account**
**Note:** Roles, ClusterRoles and RoleBindings must be configured as well

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: nginx-ingress-serviceaccount

create serviceaccount roles

    # kubectl get role --namespace ingress-nginx

    NAME                      CREATED AT
    ingress-nginx             2023-08-15T10:25:53Z
    ingress-nginx-admission   2023-08-15T10:25:53Z

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

create serviceaccount rolebindings

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

create serviceaccount clustrerrole

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

##### Ingress Controller Deployment

create controller deployment

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

##### Ingress Service

create ingress service

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

#### Ingress Resources

see:
[kubernetes.io - create ingress command ref](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-)
[kubernetes.io - ingress](https://kubernetes.io/docs/concepts/services-networking/ingress)
[kubernetes.io - ingress examples](https://kubernetes.github.io/ingress-nginx/examples/)

**Note:** ingress resource apiVersion may change depending on k8s' version

**Basic routing components**

- Single/Mutliple URLs
- Single/Mutliple paths
- Single/Mutliple backend services

get ingress resources

    kubectl get ingress --namespace <namespace-name>

create ingress imperatively

    # format
    kubectl create ingress <ingress-name> --rule="host/path=service:port"

    # example
    kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

##### Single URL - Single paths - Single backend

schema

    # URL: www.my-online-store.com
        Path: /
        backend sevice: wear-service

ingress object sample

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-wear

    spec:
      backend:
        name: wear-service
        port:
          number: 80

##### Single URL - Mutliple paths - Mutliple backends

schema

    # URL: www.my-online-store.com

        Path: /wear
        backend service: wear-service

        Path: /watch
        backend service: watch-service

        # default 404 page
        Path: *
        backend service: default-http-backend

ingress object sample

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-wear-watch

    spec:
      rules:
      - http:
          paths:
          - path: /wear
            backend:
              service:
                name: wear-service
                port:
                  number:  80

          - path: /watch
            backend:
              service:
                name: watch-service
                port:
                  number:  80

          # default 404 page
          - path: /
            backend:
              service:
                name: default-http-backend
                port:
                  number:  80

##### Multiple URLs - Mutliple backends

schema

    # URL: www.wear.my-online-store.com

        Path: *
        backend service: wear-service

    # URL: www.watch.my-online-store.com

        Path: *
        backend service: watch-service

ingress object sample

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-wear-watch

    spec:
      rules:
      - host: www.wear.my-online-store.com
        http:
          paths:
          - backend:
              service:
                name: wear-service
                port:
                  number:  80

      - host: www.watch.my-online-store.com
        http:
          paths:
          - backend:
              service:
                name: watch-service
                port:
                  number:  80

          # default 404 page
          - path: /
            backend:
              service:
                name: default-http-backend
                port:
                  number:  80

### NetworkPolicy

demo/lab image: kodekloud/webapp-conntest

- port-based or IP-based trafic control
- specify how a pod is allowed to communicate with network "entities"
- applies to connections to and from a pod

Network solutions that **support network policy:**

- Kube-router
- Calico
- Romana
- Weave-net

Network solutions that **do not support network policy:**

- Flannel
  if network policy is created, no error message will be displayed, the net policy will simply not work

network policy sample

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: test-network-policy
      namespace: default

    spec:
      podSelector:
        matchLabels:
          role: db
      policyTypes:
        - Ingress
        - Egress
      ingress:
        - from:
            - ipBlock:
                cidr: 172.17.0.0/16
                except:
                  - 172.17.1.0/24
            # And applied here because both rules
            # are in the same from element
            - namespaceSelector:
                matchLabels:
                  env: prod
              podSelector:
                matchLabels:
                  role: frontend
          ports:
            - protocol: TCP
              port: 6379
      egress:
        - to:
            - ipBlock:
                cidr: 10.0.0.0/24
          ports:
            - protocol: TCP
              port: 5978

### Port Forwarding

Environment

    kubectl get service mongo
      NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
      mongo   ClusterIP   10.96.41.183   <none>        27017/TCP   11s

    kubectl get pods
      NAME                     READY   STATUS    RESTARTS   AGE
      mongo-75f59d57f4-4nd6q   1/1     Running   0          2m4s

    kubectl get pod mongo-75f59d57f4-4nd6q --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
      27017

_`27017` is the TCP port allocated to `mongo` pod_

Forward a local port to a port on the Pod

    kubectl port-forward mongo-75f59d57f4-4nd6q       28015:27017
                         pods mongo-75f59d57f4-4nd6q  28015:27017
                         deployment mongo             28015:27017
                         replicaset mongo-75f59d57f4  28015:27017
                         service mongo                28015:27017

      Forwarding from 127.0.0.1:28015 -> 27017
      Forwarding from [::1]:28015 -> 27017

let kubectl choose the local port

    kubectl port-forward deployment/mongo :27017
      Forwarding from 127.0.0.1:63753 -> 27017
      Forwarding from [::1]:63753 -> 27017

## Section 8: State Persistance

### Volume

a directory, accessible to the containers in a pod.

Types of volumes:

- nfs
- cephfs
- configMap
- Cloud services (AWS, Azure, GCP, etc.)
- hostPath
- emptyDir
- iscsi
- local

see: [kubernetes.io/docs - Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

hostPath volume mount sample

    apiVersion: v1
    kind: Pod
    metadata:
      name: random-num-gen
    spec:
      containers:
      - image: alpine
        name: alpine
        command: ["bin/sh", "-c"]
        args: ["shuf -i 0-100 -n 1 >> /opt/number.out]
        volumeMounts:
        - mountPath: /opt
          name: data-volume
      volumes:
      - name: data-volume
        hostPath:
          path: /data
          type: Directory

### PersistentVolume

after PVCs are done, persistent volumes act upon `persistentVolumeReclaimPolicy`:

- `Retain` (default)
  keep PV, keep contents
- `Delete`
  delete PV, delete contents
- `Recycle`
  keep PV, delete contents

> For dynamically provisioned PVs, default reclaim policy is `Delete`

PV sample

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-vol
    spec:
      accessModes:
        - ReadWriteOnce
      capacity:
        storage: 1Gi
      persistentVolumeReclaimPolicy: Recycle

      hostPath:                   # node storage
        path: /tmp/data

      awsElasticBlockStore:       # AWS storage (deprecated)
        volumeID: <colume-id>
        fsType: ext4

### PersistentVolumeClaim

**Usage:**

- define PV
- create PVC
- use PVC in `Pod`, `ReplicaSet` or `Deployment`

PVCs can specify a `label` selector to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim.
The selector can consist of two fields:

- `matchLabels`
  the volume must have a label with this value
- `matchExpressions`
  a list of requirements made by specifying key, list of values, and operator that relates the key and values. Valid operators include `In`, `NotIn`, `Exists`, and `DoesNotExist`.

> requirements from both `matchLabels` and `matchExpressions` must all be satisfied to match

**PVC AccessModes:**

- `ReadWriteOnce`
- `ReadOnlyMany`
- `ReadWriteMany`

PVC sample

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: vol-claim-1
    spec:
      accessModes:
        - ReadWriteOnce
      resourcecs:
        requests:
          storage: 500Mi

use pvc in pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      containers:
        - name: myfrontend
          image: nginx
          volumeMounts:
          - mountPath: "/var/www/html"
            name: pvc-vol
      volumes:
        - name: pvc-vol
          persistentVolumeClaim:
            claimName: vol-claim-1

### StorageClass

Dynamic provisioning of volumes; upon a claim, the storage is created.

StorageClass sample

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: google-storage
    provisioner: kubernetes.io/gce-pd
    volumeBindingMode: [ ImmediateWaitForFirstConsumer ]
    parametes:
      types: [ pd-standard | pd-ssd ]
      replication-type: [ none | regional-pd ]

use StorageClass in PVC

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: vol-claim-1
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: google-storage
      resourcecs:
        requests:
          storage: 500Mi

use PVC in pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: random-num-gen
      ...
      volumes:
      - name: data-volume
        persistentVolumeClaim:
            claimName: vol-claim-1

### StatefulSets

for applications that require one or more of the following:

- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

allows you to relax ordering guarantees with `.spec.podManagementPolicy` field it can be:

- `OrderedReady` (default).
- `Parallel`

> StatefulSets **must** be configured with a headless service for DNS naming

headless service and stateful set sample

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx

    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web
    spec:
      selector:
        matchLabels:
          app: nginx # has to match .spec.template.metadata.labels
      serviceName: "nginx"
      replicas: 3 # by default is 1
      template:
        metadata:
          labels:
            app: nginx # has to match .spec.selector.matchLabels
        spec:
          containers:
          - name: nginx
            image: registry.k8s.io/nginx-slim:0.8
            ports:
            - containerPort: 80
              name: web
            volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: www
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "my-storage-class"
          resources:
            requests:
              storage: 1Gi

### Headless Services

- have NO load-balancing
- have NO single Service IP address
- used for service discovery mechanisms (DNS names)

headless service sample

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx

subscribe a pod to a headless service

    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web

      # w/out hostname given, pod name:
      # mysql-h.default.svc.cluster.local
      subdomain: mysql-h

      # w/ subdomain, pod name:
      # mysql-pod.mysql-h.default.svc.cluster.local
      hostname: mysql-pod

### volumeClaimTemplates

`volumeClaimTemplates` provide stable storage using PVs provisioned by a PV Provisioner.

On **pod failure/reschedules, volumeClaimTemplates-v PVCs are not removed** but are **instead attached to the recreated pods**

use volumeClaimTemplates in a StatefulSet

    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web
    spec:
      selector:
        matchLabels:
          app: nginx # has to match .spec.template.metadata.labels
      ...
      volumeClaimTemplates:
      - metadata:
          name: www
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "my-storage-class"
          resources:
            requests:
              storage: 1Gi

## Section 9: Post Sep-2021 Changes

### Operator Framework

#### API Maintenance

discover API tree

    curl https://localhost:8001 -k
    curl https://localhost:8001/apis -k | grep name

##### APIs hierarchy

discover API:
`curl https://my-kube-playground:6443/version`
`curl https://my-kube-playground:6443/api/v1/pods`

API Groups:

- /metrics
- /healthz
- /version
- /api
- /apis
- /logs

**named group**

- `/apis`
  **API groups:**
  - `/extensions`
  - `/storage.k8s.io`
  - `/authentication.k8s.io`
  - `/certificates.k8s.io`
  - `/networking.k8s.io`
    - `/v1`
      - `/networkpolicies`
  - `/apps`
    - `/v1`
      **resources:**
      - `/deploymens`
        **verbs**
        - `list`
        - `get`
        - `create`
        - `delete`
        - `update`
        - `watch`
      - `/replicasets`
      - `/statefulsets`

**core group**

- `/api`
  - `/v1`
    - `namespaces`
    - `pods`
    - `rc`
    - `events`
    - `endpoints`
    - `nodes`
    - `bindings`
    - `PV`
    - `PVC`
    - `configmaps`
    - `secrets`
    - `services`

##### API Versions

In Kubernetes versions : `X.Y.Z`
`X` major, `Y` minor, `Z` patch version.

get supported API versions on the server as "group/version"

    kubectl api-versions

API versions:

- **Alpha**
  _vXalphaY_ (example, _v1alpha1_).

  - disabled by default
  - audience: expret users

- **Beta**
  _vXbetaY_ (example, _v2beta3_).

  - disabled by default
  - audience: beta testers

- **stable/ GA (Generally Available)**
  _vX_ (example, _v1_).

  - enabled by default
  - audience: all-users

create pod using different versions

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      ...

    apiVersion: apps/v1alpha1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      ...

    apiVersion: apps/v1beta2
    kind: Deployment
    metadata:
      name: nginx
    spec:
      ...

**Preferred version:** default get/query version

see preferred version for api group:

    curl 127.0.0.1:8001/apis/batch | grep -iA5 preferredversion

**Storage version:** version objects are stored in in etcd (regardless of the version in definition files)

see storage version

    ETCDCTL_API=3 etcdctl \
      --endpoints=https://[127.0.0.1]:2379 \
      --cacert=<path> \
      --cert=<path> \
      --key=<path> \
      get "/registry/deployment/default/<deployment-name> --print-value-only

enable/disable API group

    ExecStart=/usr/local/bin/kube-apiserver \\
      ...
      --runtime-config=batch/v2alpha1,... \\
      ...

##### API Deprecation

**Depreation Rules:**

- **Rule #1:** API elements may only be removed by incrementing the version of the API group.
- **Rule #2:** API objects must be able to round-trip between API versions in a given release without information loss, with the exception of whole REST resources that do not exist in some versions.
- **Rule #3:** An API version in a given track may not be deprecated in favor of a less stable API version.
- **Rule #4a:** API lifetime is determined by the API stability level

  - GA API versions may be marked as deprecated, but must not be removed within a major version of Kubernetes
  - Beta API versions are deprecated no more than 9 months or 3 minor releases after introduction (whichever is longer), and are no longer served 9 months or 3 minor releases after deprecation (whichever is longer)
  - Alpha API versions may be removed in any release without prior deprecation notice

- **Rule #4b:** The "preferred" API version and the "storage version" for a given group may not advance until after a release has been made that supports both the new version and the previous version

bulk convert definition files form a version to another

    # need to install the convert plugin
    kubectl convert -f nginx_def.yaml --output-version <new-api>

- install kubectl-convert:
  https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin

#### CustomResourceDefinition

Represents a custom current/desired state of k8s resource.

CRD/custom resource sample 1

    apiVersion: flights.com/v1
    kind: FlightTicket
    metadata:
      name: myflight-ticket
    spec:
      from: Mumbai
      to: London
      number: 2

    ---
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
      name: flighttickets.flights.com

    spec:
      scope: Namespaced
      # api group
      group: flights.com
      names:
        kind: FlightTicket
        singular: flightticket
        plural: flighttickets
        shortNames:
          - ft

      versions:
        - name: v1
          served: true    # preferred version
          storage: true   # storage version

      schema:
        openAPIV3Schema:
          type: Object
          properties:
            spec:
              type: Object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
                  maximum: 10

CRD/custom resource sample 2

    apiVersion: traffic.controller/v1
    kind: Global
    metadata:
      name: datacenter
    spec:
      dataField: 2
      access: true

    ---
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
      name: globals.traffic.controller
    spec:
      conversion:
        strategy: None
      group: traffic.controller
      names:
        kind: Global
        listKind: GlobalList
        plural: globals
        shortNames:
        - gb
        singular: global
      scope: Namespaced
      versions:
      - name: v1
        schema:
          openAPIV3Schema:
            properties:
              spec:
                properties:
                  access:
                    type: boolean
                  dataField:
                    type: integer
                type: object
            type: object
        served: true
        storage: true

#### CustomControllers

Customly keep the state of Kubernetes objects in sync with declared desired states.

sample controller: `https://github.com/kubernetes/sample-controller`

#### Operators

**Operators** = **Custom Controllers** + **Custom Resource Definitions (Crds)**

**Community operators:** https://operatorhub.io

see: [kubernetes.io - An example operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/#example)

example

    Custom Resource Definition (CRD)      Custom Controller
    ------------------------------------------------------------
    EtcdCluster                           ETCD Controller
    EtcdBackup                            Backup Operator
    EtcdRestore                           Restore Operator

### Deployment Strategies

#### Blue Green Deployments

Steps:

1. create deployment v1 (blue)
2. create deployment service
3. create deployment v2 (green)
4. perform tests on v2 (optional)
5. switch service label selector to v2 (green)

#### Canary Deployments

Steps:

1. create deployment v1 (blue)
2. create deployment service
3. create deployment v2 (green)
4. route small percentage fo trafic to v2 (green)
   4a. route traffic to both versios
   use a common label
   4b. route a small % of trafic to v2
   reduce num of pods in v2 (green)
5. if no issues, switch service label selector to v2 (green)

### Helm

Automates the creation, packaging, configuration, and deployment of Kubernetes applications by combining definition files into a single reusable package.

Installation pre-reqs:

- k8s cluster
- kubectl (configured)

install helm
see: https://helm.sh/docs/intro/install

    # using snap
    sudo snap install helm --classic

    # using package manager
    sudo dnf install helm
    sudo apt-get install helm

check helm installation

    helm version

    # get helm client env information
    helm env

define helm variables

    cat values.yaml
      image: wordpress:4.8-apache
      storage: 20Gi
      passwordEncoded: DkfEhMs.....

use helm variables in a `values.yaml`

    cat templates/pv.yaml
      ...
      {{ .Values.storage }}
      ...

**Helm Charts** = **Helm Variables** + **Helm Templates** + **Chart.yaml** (chart meta infos)

**Helm Repos:**

- ArtifactHub
  https://artifacthub.io
- Bitnami
  https://charts.bitnami.com/

helm repo commands

    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm search hub wordpress
    helm repo list

install helm chart (download, extract, install locally)

    # release-name: chart installation
    helm install [release-name] [chart-name]

    helm install release-1 bitnami/wordpress
    helm install release-2 bitnami/wordpress
    helm install release-3 bitnami/wordpress

manage helm releases and charts

    helm list
    helm uninstall <release-name>
    helm pull --untar bitnami/wordpress

## Section 10: Labs

- Kubernetes Challenges
  https://kodekloud.com/courses/kubernetes-challenge

- KLLR SHLL - Linux Foundation Exam Simulators
  https://killer.sh

- KLLR CODA - Interactive environments
  https://killercoda.com
