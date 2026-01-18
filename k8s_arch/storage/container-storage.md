# Container Storage

tags: #storage

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Container Storage](#container-storage)
  - [Docker Storage](#docker-storage)
  - [Kubernetes Container-Runtime Storage](#kubernetes-container-runtime-storage)
    - [Container Storage Interface](#container-storage-interface)

<!-- /code_chunk_output -->

---

## Docker Storage

filesystem **storage paths:**

```
/var/lib/docker
├── aufs
├── containers
├── image
└── volumes # volume mounts
    ├── my_data1
    └── my_data2
```

**Docker Filesystem Architecture**

> managed by storage drivers

- **Container Layer**
  - ephemeral
  - read/write
  - when modifying "read-only" files:
    - **copy-on-write** mechanism
      - copies file to container layer
- **Image Layer**
  - layered
  - read only
- **Mounts**
  - **Volume Mounts**
    - path: `/var/lib/docker/volumes`
  - **Bind Mounts**
    - host filesystem paths, example: `/srv/containers/..`

**Docker Storage Drivers**

- manages: **Bind Mounts**
- depends on underlying OS
- drivers
  - AUFS (Ubuntu)
  - ZFS
  - BTRFS
  - Device Mapper (Fedora, CentOS)
  - Overlay
  - Overlay2

**Docker Volume Drivers**

- manages: **Volume Mounts**
- drivers:
  - Local
  - Azure File Storage
  - Convoy
  - DigitalOcean Block Storage
  - Flocker
  - gce-docker
  - GlusterFS
  - NetApp
  - RexRay (support AWS EBS, S3, Google Persistant Disk)
  - Portworx
  - VMware vSphere Storge

## Kubernetes Container-Runtime Storage

### Container Storage Interface

- defines **Remote Procedure Calls (RPCs)**
  - request flow: orchestrator > Runtime
    - SHOULD be called by orchestrators
    - SHOULD be implemented by drivers/plugins
  - **RPCs define:**
    - params sent by caller and received by solution
    - error codes to be exchanged
  - **RPCs:**
    - `CreateVolume`: to provision new volumes
    - `DeleteVolume`: to decomission a volume
    - `ControllerPublish `: to make the volume available on a node
