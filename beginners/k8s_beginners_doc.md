#k8s For the Absolute Beginners

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [k8s For the Absolute Beginners](#k8s-for-the-absolute-beginners)
  - [Documentation](#documentation)
  - [Architecture](#architecture)
    - [Controllers](#controllers)
      - [ReplicationController](#replicationcontroller)
      - [ReplicaSet](#replicaset)
      - [Deployment](#deployment)
    - [Networking](#networking)
    - [Services](#services)
      - [NodePort](#nodeport)
      - [ClusterIP](#clusterip)
      - [LoadBalancer](#loadbalancer)

<!-- /code_chunk_output -->

## Documentation

[github.com - Kubernetes](https://github.com/kubernetes/kubernetes)

[kubernetes.io](https://kubernetes.io)

[kubernetes.io - Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

[opensource.com - A guide to Kubernetes architecture](https://opensource.com/article/22/2/kubernetes-architecture)

[opensource.com - A visual guide to Kubernetes networking fundamentals](https://opensource.com/article/22/6/kubernetes-networking-fundamentals?utm_medium=Email&utm_campaign=weekly&sc_cid=7013a00000311fXAAQ)

[redhat.com - How Kubernetes creates and runs containers: An illustrated guide](https://www.redhat.com/architect/how-kubernetes-creates-runs-containers)

[opensource.com - A visual map of a Kubernetes deployment](https://opensource.com/article/22/3/visual-map-kubernetes-deployment)

[medium.com - Scaling Kubernetes to Over 4k Nodes and 200k Pods](https://medium.com/paypal-tech/scaling-kubernetes-to-over-4k-nodes-and-200k-pods-29988fad6ed)

[kompose.io - DOCKER COMPOSE TO KUBERNETES](https://kompose.io)

[opensource.com - Migrate databases to Kubernetes using Konveyor](https://opensource.com/article/22/5/migrating-databases-kubernetes-using-konveyor)

[github.com/dockersamples - docker sample apps](https://github.com/dockersamples)

[Dockerhub - kodekloud voting app images](https://hub.docker.com/r/kodekloud/examplevotingapp_worker)

voting app

- https://github.com/kodekloudhub/example-voting-app

- https://github.com/kodekloudhub/example-voting-app-kubernetes


## Architecture

**k8s components:**

- **api-server**
- **etcd**
- **scheduler**
- **controller manager**
  Controllers are loops that watch the state of your cluster, then make or request changes where needed.

  Controllers tracks **at least** one Kubernetes resource type, and these objects have a spec field that represents the **desired state**. Controllers use the **watch mechanism** to get notified of changes. They watch the API server for changes to resources and perform operations for each change.

  The Controller Manager also performs lifecycle functions such as namespace creation and lifecycle, event garbage collection, terminated-pod garbage collection, cascading-deletion garbage collection, and node garbage collection.
  See [Cloud Controller Manager for more information](https://kubernetes.io/docs/concepts/architecture/cloud-controller/).

- **kubelet**
- **container runtime**

**controle node components:**

- api-server
- etcd
- scheduler
- controller manager

**worker node components:**

- kubelet
- container runtime

### Controllers

#### ReplicationController

Ensures that a specified number of pod replicas are running at any one time.
Note: `selector` field is not required, if skipped it assumes the same labels in the pod definition

#### ReplicaSet

Maintains a stable set of replica Pods running at any given time. Can manage pods not started by it.

**Notes:**

- A ReplicaSet is linked to its Pods via the Pods' `metadata.ownerReferences` field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their `ownerReferences` field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.

  A ReplicaSet identifies new Pods to acquire by using its `selector`. If there is a Pod that has no `OwnerReference` or the `OwnerReference` is not a ` ` and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

- When a replicaset is deleted via `kubectl delete` its pods are also deleted

#### Deployment

The following are typical use cases for Deployments:

- **Create a Deployment to rollout a ReplicaSet**. The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
- **Declare the new state of the Pods** by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
- **Rollback to an earlier Deployment revision** if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
- **Scale up the Deployment** to facilitate more load.
- **Pause the rollout of a Deployment** to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
- **Use the status of the Deployment** as an indicator that a rollout has stuck.
- **Clean up older ReplicaSets** that you don't need anymore.

**Deployment strategies:**

- **recreate**
  destroys all old app version instances **then** launch new one
- **rolling update**
  **default strategy**, recreate instances **few-at-a-time**

**Note:**
Adding the `--record` option to _deployment creation and modification commands_ allows the recording of change cause in the revision history

### Networking

- Internal Private Network is created when k8s is created/configured
- Each pod has an ip address

**Kubernetes Fundamental Networking Requirements:**

- **Containers or PODs** in a kubernetes cluster MUST be able to **communicate without configuring NAT**.

- **All nodes** must be able to **communicate with containers** and **all containers** must be able to **communicate with the nodes** in the cluster.

### Services

Enable communication between various components within and
outside of the application.

Main types of services:

- **NodePort**
- **ClusterIP**
- **LoadBalancer**

#### NodePort

Makes an **internal POD** accessible on a **Port on the Node**. 
By default, **load is balanced** across multiple nodes **randomly with session affinity**.

Ports:

- **TargetPort**: pod port
- **Port**: service port
- **NodePort**: node port; valid range: 30000 - 32767

Finds **pods with labels matching** those defined in the **service selector section**.

#### ClusterIP

Creates a **virtual IP inside the cluster** to enable **communication between different services**.

#### LoadBalancer

Provisions a load balancer for our service in supported cloud providers.
