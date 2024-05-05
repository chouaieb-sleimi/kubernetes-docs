v# K8S Observability & Monitoring

tags: #api

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Global Object List](#global-object-list)
- [API Maintenance](#api-maintenance)
  - [API hierarchy](#api-hierarchy)
    - [API Versioning](#api-versioning)
  - [API Deprecation](#api-deprecation)

<!-- /code_chunk_output -->

---

## Global Object List

[[objects_list]]

## API Maintenance

discover API tree

```bash
curl https://api-server-url:8001 -k
curl https://api-server-url:8001/apis -k | grep name
curl https://my-kube-playground:6443/version
curl https://my-kube-playground:6443/api/v1/pods
```

### API hierarchy

[[api_groups]] hierarchy:

- `/metrics`
- `/healthz`
- `/version`
- `/logs`
- `/api` (core group)
  - `v1`
    - `resources`
      - `verbs`
- `/apis` (named groups)
  - `named group`
    - `version`
      - `resources`
        - `verbs`

#### API Versioning

In Kubernetes versions : `X.Y.Z`. This versions all components: apiserver, kubelet, kubectl, etc.
`X` major, `Y` minor, `Z` patch version.

get supported API versions on the server as "group/version"

    kubectl api-versions

API versions:

- **Alpha**
  _vXalphaY_ (example, _v1alpha1_).

  - disabled by default
  - audience: expret users

- **Beta**
  _vXbetaY_ (example, _v2beta3_).

  - disabled by default
  - audience: beta testers

- **stable/ GA (Generally Available)**
  _vX_ (example, _v1_).

  - enabled by default
  - audience: all-users

create pod using different versions

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      ...

    apiVersion: apps/v1alpha1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      ...

    apiVersion: apps/v1beta2
    kind: Deployment
    metadata:
      name: nginx
    spec:
      ...

**Preferred version:** default get/query version

see preferred version for api group:

    curl 127.0.0.1:8001/apis/batch | grep -iA5 preferredversion

**Storage version:** version objects are stored in in etcd (regardless of the version in definition files)

see storage version (must have `etcdctl` installed)

    ETCDCTL_API=3 etcdctl \
      --endpoints=https://[127.0.0.1]:2379 \
      --cacert=<path> \
      --cert=<path> \
      --key=<path> \
      get "/registry/deployment/default/<deployment-name> --print-value-only

enable/disable API group

    ExecStart=/usr/local/bin/kube-apiserver \\
      ...
      --runtime-config=batch/v2alpha1,... \\
      ...

### API Deprecation

**Depreation Rules:**

- **Rule 1:** API elements may only be removed by incrementing the version of the API group.
- **Rule 2:** API objects must be able to round-trip between API versions in a given release without information loss, with the exception of whole REST resources that do not exist in some versions.
- **Rule 3:** An API version in a given track may not be deprecated in favor of a less stable API version.
- **Rule 4a:** API lifetime is determined by the API stability level

  - GA API versions may be marked as deprecated, but must not be removed within a major version of Kubernetes
  - Beta API versions are deprecated no more than 9 months or 3 minor releases after introduction (whichever is longer), and are no longer served 9 months or 3 minor releases after deprecation (whichever is longer)
  - Alpha API versions may be removed in any release without prior deprecation notice

- **Rule 4b:** The "preferred" API version and the "storage version" for a given group may not advance until after a release has been made that supports both the new version and the previous version

bulk convert definition files form a version to another

    # need to install the convert plugin
    kubectl convert -f nginx_def.yaml --output-version <new-api>

- install kubectl-convert:
  https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin

