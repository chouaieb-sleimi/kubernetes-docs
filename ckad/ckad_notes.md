# Certified Kubernetes Application Developer - CKAD

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=3 orderedList=false} -->

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
    - [Secrets](#secrets)
    - [Security](#security)
    - [ServiceAccounts](#serviceaccounts)
    - [Resource Requirements](#resource-requirements)
    - [ResourceQuota](#resourcequota)
    - [Taints and Tolerations](#taints-and-tolerations)
    - [Node Selectors and Affinity](#node-selectors-and-affinity)
  - [Section 4: Multi-Container Pods](#section-4-multi-container-pods)
    - [Init Containers](#init-containers)
  - [Section 5: Observability](#section-5-observability)
    - [Readiness and Liveness Probes](#readiness-and-liveness-probes)
    - [Readiness Probe](#readiness-probe)
    - [Liveness Probe](#liveness-probe)
    - [Container Logging](#container-logging)
    - [Monitoring Cluster](#monitoring-cluster)
  - [Section 6: Pod Design](#section-6-pod-design)
    - [Labels Selectors and Annotations](#labels-selectors-and-annotations)
    - [Rolling Updates and Rollbacks in Deployments](#rolling-updates-and-rollbacks-in-deployments)
    - [Jobs and CronJobs](#jobs-and-cronjobs)
  - [Section 7: Services and Networking](#section-7-services-and-networking)
    - [Ingress](#ingress)
    - [Network Policies](#network-policies)
  - [Section 8: State Persistance](#section-8-state-persistance)
    - [Volume](#volume)
    - [PersistentVolume](#persistentvolume)
    - [PersistentVolumeClaim](#persistentvolumeclaim)
    - [StorageClass](#storageclass)
    - [StatefulSets](#statefulsets)
    - [Headless Services](#headless-services)
    - [volumeClaimTemplates](#volumeclaimtemplates)
    - [Section 9: Post Sep-2021 Changes](#section-9-post-sep-2021-changes)

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

create sa

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

- are appplicable on the namespace level
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

Are used to set **restrictions on what pods nodes _can (and not must)_ accept**.

#### Taints (Node)

see a node's taints

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

Constrain a Pod so that it is restricted to run on particular node(s), or to prefer to run on particular nodes.

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

## Section 5: Observability

### Readiness and Liveness Probes

**Failing liveness probe will restart the container**, whereas **failing readiness probe will stop our application from serving traffic**.

demo image: kodekloud/webapp-delayed-start

#### Pod Status

**Pod States:**

- Pending (Pod not scheduled)
- ContainerCreating
- Running

get pod status

    kubectl get pods
    kubectl describe pod <pod-name> | grep -i status

#### Pod Conditions

Pod conditions compliment pod status. Can be true or false.

**Pod conditions:**

- PodScheduled
- Initialized
- ContainerReady (containers are running)
- Ready (pod is running)

get pod conditions

    kubectl describe pod <pod-name> | grep -iA5 conditions

### Readiness Probe

**Readiness probe types:**

- HTTP request test
- Port test
- Script execution

set a **container readiness check**

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

### Liveness Probe

Defines **when an application in a conatainer is healthy**

**Liveness probe types:**

- HTTP request test
- Port test
- Script execution

set a **acontainer liveness check**

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

### Container Logging

demo image: kodekloud/event-simulator

**get a container's logs** (`<pod-container>` are necessary for multi-container pods.)

    kubectl logs -f <pod-name> <container-name>

### Monitoring Cluster

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

K8s monitoring solutions:

- heapster **(deprecated)**
- metric server
- prometheus
- ELK stack
- data dog (proprietary)
- dynatracee (proprietary)

#### Metrics server Overview

Is a slimmed down version of heapster. can only be 1 metrics server per k8s cluster. An **In-Memory monitoring solution**; doesn't store logs and mterics data on disk.

Uses a `kubelet` component `cAdvisor`; retrieves pod performance metrics and expose them through kubelet api to metrics server.

#### Metrics Server Deployment

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

**Labels** are intended to be used to specify **identifying attributes of objects** that are meaningful and relevant to users, but do not directly imply semantics to the core system.
Labels **can be used to organize and to select subsets of objects**.

Via a label **selector**, the client/user **can identify a set of objects**.
The API currently supports **two types of selectors: equality-based and set-based**. A label selector can be made of multiple requirements which are comma-separated.

You can use Kubernetes **annotations** to attach **arbitrary non-identifying metadata** to objects. Clients such as tools and libraries can retrieve this metadata.

### Rolling Updates and Rollbacks in Deployments

Rollout strategies:

- Recreate strategy
- Rolling update (default strategy)

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

demo image: kodekloud/throw-dice

#### Jobs

Creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate.

Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

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
      template:
        spec:
          containers:
          - name: math-add
            image: ubuntu
            command: ['expr', '3', '+', '2']
          restartPolicy: Never
      backoffLimit: 4   # default is 6

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

Enable communication between various components within and
outside of the application.

Main types of services:

- **NodePort**
- **ClusterIP**
- **LoadBalancer**

### Ingress

**demo images:**

- kodekloud/ecommerce:apparels
- kodekloud/ecommerce:video
- kodekloud/ecommerce:food
- kodekloud/ecommerce:404

Exposes HTTP and HTTPS **routes from outside the cluster to services within the cluster.** Traffic routing is controlled by rules defined on the Ingress resource.
Ingress may provide load balancing, SSL termination and name-based virtual hosting.

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

**Deployment resources/steps:**

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

    #k get role --namespace ingress-nginx

    NAME                      CREATED AT
    ingress-nginx             2023-08-15T10:25:53Z
    ingress-nginx-admission   2023-08-15T10:25:53Z

    #k get role ingress-nginx --namespace ingress-nginx -o yaml
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

    #k get role ingress-nginx-admission --namespace ingress-nginx -o yaml
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

          # nginx service needs these to read config data from within the pod
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

Routing schema

    # URL: www.my-online-store.com
        Path: /
        backend sevice: wear-service

create ingress object

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-wear

    spec:
      backend:
        name: wear-service
        port:
          number: 80

##### Mutliple paths - Mutliple backends

Single rule, multiple paths each

Routing schema

    # URL: www.my-online-store.com

        Path: /wear
        backend service: wear-service

        Path: /watch
        backend service: watch-service

        # default 404 page
        Path: *
        backend service: default-http-backend

create ingress resource

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

Multiple rules, single path each

Routing schema

    # URL: www.wear.my-online-store.com

        Path: *
        backend service: wear-service

    # URL: www.watch.my-online-store.com

        Path: *
        backend service: watch-service

create ingress resource

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

### Network Policies

demo/lab image: kodekloud/webapp-conntest

Control traffic flow at the IP address or port level. Allows you to specify how a pod is allowed to communicate with various network "entities" over the network.
NetworkPolicies apply to a connection with a pod on one or both ends, and are not relevant to other connections.

Network solutions that **support network policy:**

- Kube-router
- Calico
- Romana
- Weave-net

Network solutions that **do not support network policy:**

- Flannel
  if net policy created no error message will be displayed, the net policy will simply not work

create network policy

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

## Section 8: State Persistance

### Volume

Is a directory, possibly with some data in it, which is accessible to the containers in a pod.

How that directory comes to be, the medium that backs it, and the contents of it are determined by the particular volume type used.

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

create and mount hostPath volume

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

after claims, persistent volumes act upon `persistentVolumeReclaimPolicy` which can be `Retain` (default), `Delete` or `Recycle`

create persistent volume

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-vol
    spec:
      accessModes:
        - ReadWriteOnce
      capacity:
        storage: 1Gi

      # node storage
      hostPath:
        path: /tmp/data

      # or AWS (deprecated)
      awsElasticBlockStore:
        volumeID: <colume-id>
        fsType: ext4

### PersistentVolumeClaim

**Usage:**

- Define PV
- Create PVC
- Use PVC in Pods, ReplicaSets or Deployments

Claims can specify a `label` selector to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim.
The selector can consist of two fields:

- `matchLabels` - the volume must have a label with this value
- `matchExpressions` - a list of requirements made by specifying key, list of values, and operator that relates the key and values. Valid operators include `In`, `NotIn`, `Exists`, and `DoesNotExist`.

All of the requirements, from both matchLabels and matchExpressions, are ANDed together – **they must all be satisfied in order to match.**

**PVC AccessModes:**

- ReadWriteOnce
- ReadOnlyMany
- ReadWriteMany

persistent volume claim example

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

Does dynamic provisioning of volumes; upon a claim, the storage is created.

StorageClass definition

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: google-storage
    provisioner: kubernetes.io/gce-pd
    parametes:
      types: [ pd-standard | pd-ssd ]
      replication-type: [ none | regional-pd ]

use StorageClass in PersistentVolumeClaim

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

use persistentVolumeClaim in pod

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

StatefulSets are valuable for applications that require one or more of the following.

- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

StatefulSet allows you to relax ordering guarantees with `.spec.podManagementPolicy` field; it can be `Parallel` or `OrderedReady` (default).

**Note:** StatefulSets must be configured with a headless service

headless service and stateful set definitions

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

Are services that

- have NO load-balancing
- have NO single Service IP address
- used for service discovery mechanisms (DNS names)

headless service definition

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

volumeClaimTemplates will provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner.

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

### Section 9: Post Sep-2021 Changes
