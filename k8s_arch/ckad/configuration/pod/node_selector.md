# K8S Node Selectors

tags: #configuration

---

- k8S only schedules the [[pod]] onto nodes that have each of the labels specified.

create **node label to be used as a selector** for pods

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

run pod on selected nodes
```yaml
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
```
