# K8S iscsi Volume

tags: #storage

---

- mount an existing iSCSI (SCSI over IP)
- can be mounted by **multiple read-only** consumers,
  - can only be mounted by a **single read-write** consumer.
- when a **pod is removed,**
  - volume **contents are preserved,**
    - **volume unmounted.**
