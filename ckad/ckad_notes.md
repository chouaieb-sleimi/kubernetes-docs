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
    - [Commands And Arguments](#commands-and-arguments)
    - [ConfigMap](#configmap)
      - [Intro: Environment Variables](#intro-environment-variables)
      - [Create ConfigMap](#create-configmap)
      - [Use ConfigMap](#use-configmap)
    - [Secrets](#secrets)
      - [Create Secrets](#create-secrets)
      - [Use Secret](#use-secret)
    - [Security](#security)
      - [Docker Security](#docker-security)
      - [SecurityContexts](#securitycontexts)
    - [ServiceAccounts](#serviceaccounts)
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
  - [Section 4: Observability](#section-4-observability)
    - [Readiness and Liveness Probes](#readiness-and-liveness-probes)
      - [Pod Status](#pod-status)
      - [Pod Conditions](#pod-conditions)
    - [Readiness Probe](#readiness-probe)
    - [Liveness Probe](#liveness-probe)

<!-- /code_chunk_output -->

# Section 1: Overview

## Resources

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

### Commands And Arguments

`IMAGE/entrypoint = K8S/command` and `IMAGE/cmd = K8S/args`

**Containerfile**

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

used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as:

- environment variables,
- command-line arguments,
- or as configuration files in a volume.

A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

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

- **imperative approach**

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

**inject configmap**

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

**inject single variable**

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

**inject configmap from volume**

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

demo image: kodekloud/simple-webapp-mysql

Contains a small amount of sensitive data such as a password, a token, or a key.

Secrets are:

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

- **imperative approach**

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

**inject secret**

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

**inject single variable**

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

**inject secret from volume**

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

#### Docker Security

User capabilities

    /usr/include/linux/capability.h

Override user privileges in docker run cmd

    # add privilege flag
    docker run --cap-add MAC_ADMIN ubuntu

    # drop privilege flag
    docker run --cap-drop KILL ubuntu

    # add ALL privileges
    docker run --privileged ubuntu

#### SecurityContexts

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

### ServiceAccounts

demo image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard

Service account generates an access token in a secret object upon creation.

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

**create sa**

    kubectl create sa/jenkins-sa

create an access token in a secret object **post v1.24**:
_serviceAccount must be created first_

    # expires after 1 hour
    k create token jenkins-sa

    # doesn't expire
    apiVersion: v1
    kind: Secret
    type: kubernetes.io/service-account-token
    metadata:
      name: jenkins-sa
      annotations:
        kubernetes.io/service-account.name: jenkins-sa

**see sa token**

    kubectl describe serviceaccount jenkins-sa | grep -i token

    jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64 | fromjson' <<< <secret_token>

#### Use ServiceAccounts

**use sa token**

    curl htts://192.168.56.70:6443/api -insecure \
    --header "Authorization: Bearer <sa_access-token>"

**use sa in a pod**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
        - name:
          ...
      serviceAccountName: jenkins-sa

**disable default sa automount**

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

Compute Resources required by a container. resource types are **CPU** and **MEM**.
When exceeding **CPU**, pods are throttled.
When exceeding **MEM**, pods are terminated to free memory and are re-created because of an **OOM**.

**CPU** must be >0.1cpu or >1m (`1cpu = 1000m`; `m: milli`)
**MEM** can be 256Mi = 268 M = 268435456

possible **requests/limits scenarios:**

- **NO REQUESTS / NO LIMITS**

  - **CPU**
    pods can consume all the resources ans starve others
  - **MEM**
    pods can consume all the resources ans starve others

- **NO REQUESTS / LIMITS**

  - **CPU**
    requests = limits
  - **MEM**
    requests = limits

- **REQUESTS / LIMITS**

  - **CPU**
    requests are guaranteed, exceeding limits results in throtelling
  - **MEM**
    requests are guaranteed, exceeding limits results in recreation

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

- are appplicable on the namespace level
- default limit values for pods created without requests or limits.
- affect **only newly created pods.**

**create cpu LimitRange**

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

**create memory LimitRange**

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

- are appplicable on the namespace level
- limits total resource usage on the namespace

**create ResourceQuota**

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

Are used to set **restrictions on what pods nodes _can (and not must)_ accept**.

#### Taints (Node)

**see a node's taints**

    k describe nodes <node-name> | grep -i taint

**create a taint** on a node
`taint-effect` is what happends to PODs that DO NOT TOLERATE this taint:

- **NoSchedule**
  pods will not be scheduled on the node
- **PreferNoSchedule**
  try to avoid placing pod on node
- **NoExecute**
  new pods will not be scheduled on the node, existing pods that don't tolerate the taint are evicted

      kubcetl taint nodes <node-name> key=value:<taint-effect>

      kubcetl taint nodes node01 app=blue:NoSchedule

#### Tolerations

**add toleration to a pod**

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

Constrain a Pod so that it is restricted to run on particular node(s), or to prefer to run on particular nodes.

#### Node Selectors

with `nodeSelector`, you can add the nodeSelector field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

create **node label to be used as a selector** for pods

    kubectl label nodes <node-name> <label-key>=<label-value>

**run pod on selected nodes**

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

Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

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

**add node affinity to a pod**

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

**examlpe:**

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

Pods that run multiple containers that need to work together. A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources.

containers share lifecycle, network space (they can reference each other with `localhost`) and storage volumes.

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

A task that will be **run only one time when the pod is first created.** Or a process that waits for an external service or database to be up before the actual application starts.

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

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts.

You can configure multiple such initContainers as well. In that case each init container is run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

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

## Section 4: Observability

### Readiness and Liveness Probes

#### Pod Status

**Pod States:**

- Pending (Pod not scheduled)
- ContainerCreating
- Running

**get pod status**

    kubectl get pods
    kubectl describe pod <pod-name> | grep -i status

#### Pod Conditions

Pod conditions compliment pod status. Can be true or false.

**Pod conditions:**

- PodScheduled
- Initialized
- ContainerReady (containers are running)
- Ready (pod is running)

**get pod conditions**

    kubectl describe pod <pod-name> | grep -iA5 conditions

### Readiness Probe

**Readiness probe types:**

- HTTP request test
- Port test
- Script execution

set **a readiness check on a acontainer**

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
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 8     # default is 3

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

### Liveness Probe

Defines **when an application in a conatainer is healthy**

**Liveness probe types:**

- HTTP request test
- Port test
- Script execution

set **a liveness check on a acontainer**


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