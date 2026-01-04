# K8S API Groups

tags: #api

---

**API groups:**

- [[/metrics]]
- [[/healthz]]
- [[/version]] # cluster version
- [[/logs]] # 3rd party logging apps
- /[[api-core_group]] # called **core group**
  - `/v1` # _API versions_
    **resources:**
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
      **verbs:**
    - `list`
    - `get`
    - `create`
    - `delete`
    - `update`
    - `watch`
- /[[apis-named_groups]] # called **named groups**
  - /extensions
  - /storage.k8s.io
  - /authentication.k8s.io
  - /certificates.k8s.io
  - /networking.k8s.io
  - /apps
    - `/v1` # _API versions_
      **resources:**
      - `/replicasets`
      - `/statefulsets`
      - `/deployments`
        **verbs:**
      - `list`
      - `get`
      - ...
