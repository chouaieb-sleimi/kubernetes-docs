# Kustomize

tags: #k8s

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Kustomize](#kustomize)
  - [Architecture](#architecture)
  - [Kustomize Commands](#kustomize-commands)

<!-- /code_chunk_output -->

---

## Architecture

- base config
- overlay envs (dev,stg,prd)

`kustomization.yaml` elements:

- ressources
- transformers
- image transformers
- patches (changes to base config)
  - json6902 patch (inline/file)
  - strategic merge patch (inline/file)
- components (separate blocks to import to overlays)

final manifests = base config + overlay config

```bash
kustomize-example/
├── kustomization.yaml
├── base/
│   ├── api/
│   │   ├── nginx-deploy.yaml
│   │   ├── api-service.yaml
│   │   ├── redis-deploy.yaml
│   │   └── kustomization.yaml
│   └── db/
│       ├── db-deploy.yaml
│       ├── db-service.yaml
│       └── kustomization.yaml
├── components/
│   ├── cache/
│   │   ├── ...
│   │   └── kustomization.yaml
│   └── kafka/
│       ├── ...
│       └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   ├── configmap.yaml
    |   ├── db/
    |   │   ├── kustomization.yaml
    |   │   └── db-patch.yaml
    |   └── api/
    |       ├── kustomization.yaml
    |       └── api-patch.yaml
    ├── stg/
    │   ├── kustomization.yaml
    │   ├── configmap.yaml
    │   └── patches/
    └── prod/
        ├── kustomization.yaml
        ├── configmap.yaml
        └── patches/

```

`base/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - api/
  - db/
  - cache/
  - kafka/
secretGenerator:
  - name: db-secret
    literals:
      - username=admin
      - password=admin123

# transformers: applied to all ressources names/metadata
namespace: lab
namePrefix: KodeKloud-
nameSufix: ...
commonAnnotations:
  branch: master
commonLabels:
  company: KodeKloud

# image transformers: applied to all ressources names/metadata
images:
  - name: nginx # image name to change
    newName: haproxy
    newTag: 1.19.0

# components


# patches
patches:
  # inline patch (json6902)
  - target:
      kind: Deployment
      name: nginx-deploy
    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deploy
  - target:
      kind: Deployment
      name: redis-deploy
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
  - path: replicas-json6902-patch.yaml # separate file
    target:
      kind: Deployment
      name: kafka-deploy
  - target: # dictionary (key/value) change
      kind: Deployment
      name: redis-deploy
    patch: |-
      - op: replace
        path: /spec/tempalte/metadata/labels/component
        value: redis
      - op: add
        path: /spec/tempalte/metadata/labels/org
        value: KodeKloud
      - op: remove
        path: /spec/tempalte/metadata/labels/purpose
    - target: # list change (replace)
        kind: Deployment
        name: redis-deploy
      patch: |-
        - op: replace
          path: /spec/tempalte/spec/containers/0
          value:
            name: haproxy
            image: haproxy
    - target: # list change (add)
        kind: Deployment
        name: redis-deploy
      patch: |-
        - op: add
          path: /spec/tempalte/spec/containers/- # add at the end
          path: /spec/tempalte/spec/containers/1 # add in the 2nd order
          value:
            name: nginx
            image: nginx

  # strategic merge patch
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: kafka-deploy
      spec:
        replicas: 3
  - replicas-strategic-patch.yaml # separate file
  - label-patch.yaml # separate file (dictionary+list)

---
# replicas-json6902-patch.yaml (inline (json6902))
- op: replace
  path: /spec/replicas
  value: 5

---
# replicas-strategic-patch.yaml (strategic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-deploy
spec:
  replicas: 3

---
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-deploy
spec:
  template:
    metadata:
      labels:
        component: kafka # replace
        org: KodeKloud # add
        purpose: null # delete
    spec:
      contianers:
        - name: haproxy
          image: haproxy # replace
        - $patch: delete
          name: database
```

`dev/kustomization.yaml` example

```yaml
base:
  - ../../base
components:
  - ../../components/cache
ressources:
  - grafana-deploy.yaml

namespace: lab-dev
namePrefix: ...
nameSufix: -dev

patch: |-
  - op: replace
    path: /spec/replicas
    value: 2
```

## Kustomize Commands

```bash
# generate manifests yaml
kustomize build kustomize-example/

kustomize build kustomize-example/ | kubectl apply -f -
kubectl apply -k kustomize-example/
kubectl delete -k kustomize-example/
```
