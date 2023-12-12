# K8S Node Taint

tags: #objects #configuration

---

- are added to [[node]]
- `taint-effect` is what happends to PODs **that DO NOT TOLERATE this taint**:
  - **NoSchedule**
    pods will not be scheduled on the node
  - **PreferNoSchedule**
    try to avoid placing pod on node
  - **NoExecute**
    new pods will not be scheduled on the node, existing pods that don't tolerate the taint are evicted

**list** node taints

```bash
kubectl describe nodes <node-name> | grep -i taint
```

**create** a taint

```bash
kubcetl taint nodes <node-name> key=value:<taint-effect>

kubcetl taint nodes node01 app=blue:NoSchedule
```
