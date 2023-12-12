# K8S API Named Group: Apps

tags: #api

---

schema:

- /[[v1]]
  **resources:**
  - `/replicasets`
  - `/statefulsets`
  - `/deployments`
    **verbs:**
    - `list`
    - `get`
    - `create`
    - `delete`
    - `update`
    - `watch`