# K8S Deployment Rollouts and Rollbacks

tags: #objects #workloads

---

Rollout strategies:

- **Recreate** strategy
- **RollingUpdate** (default strategy)

operations:

- apply **rollout**

```bash
# apply yaml file
kubectl apply -f deployment_def.yaml [--record]

# edit deployment
kubectl edit deploy <deployment-name> [--record]

# via command (will not update file)
kubectl set image deploy <deployment-name> [--record] \
  nginx-container=nginx:1.9.1
```

- get **rollout status and history**

```bash
# describe deployment
kubectl describe deploy <deployment-name>

# rollout status
kubectl rollout status deploy <deployment-name> [--revision <revision-num>]

# rollout history
kubectl rollout history deploy <deployment-name> [--revision <revision-num>]
```

- **rollback** latest revision

```bash
# rollback update
kubectl rollback deploy <deployment-name>
```
