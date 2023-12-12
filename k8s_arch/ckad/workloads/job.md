# K8S Job

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Job](#k8s-job)

<!-- /code_chunk_output -->

---

- creates one or more Pods and will **retry execution until one or many successes.**
- **deleting** a Job will **clean up the Pods it created.**
- **suspending** a Job will **delete its active Pods** until the Job is resumed again.

define a pod **restart policy**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-pod

spec:
  containers:
    - name: math-add
      image: ubuntu
      command: ["expr", "3", "+", "2"]
  restartPolicy: Always # Always, Never, OnFailure
```

create job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job

spec:
  completions: 3 # retries until 3 successful completions
  parallelism: 3
  backoffLimit: 4 # default is 6
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      restartPolicy: Never
```
