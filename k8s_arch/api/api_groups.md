# K8S API Groups

tags: #api

---

**API groups:**

- [[/metrics]]
- [[/healthz]]
- [[/version]]
- [[/logs]]
- /[[api]]
  **API versions:**
  - `/v1`
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
- /[[apis]]
  **API named groups:**
  - /extensions
  - /storage.k8s.io
  - /authentication.k8s.io
  - /certificates.k8s.io
  - /networking.k8s.io
  - /apps
    **API versions:**
    - `/v1`
      **resources:**
      - `/replicasets`
      - `/statefulsets`
      - `/deployments`
        **verbs:**
        - `list`
        - `get`
        - ...