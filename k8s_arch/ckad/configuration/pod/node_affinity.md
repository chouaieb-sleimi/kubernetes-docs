# K8S Node Affinity

tags: #objects #configuration

---

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

conceptually similar to nodeSelector, allowing you to constrain which nodes your [[pod]] can be scheduled on based on node labels. There are two types of node affinity:

- **requiredDuringSchedulingIgnoredDuringExecution**:
  The scheduler can't schedule the Pod unless the rule is met. This **functions like nodeSelector, but with a more expressive syntax.**

- **preferredDuringSchedulingIgnoredDuringExecution**:
  The scheduler tries to find a node that meets the rule. **If a matching node is not available, the scheduler still schedules the Pod.**

Possible node affinities:

```
DuringScheduling      DuringExecution
-----------------------------------------
Required              Ignored
Preferred             Ignored
```

add node affinity to a pod

```yaml
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
```

example:

```yaml
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
```
