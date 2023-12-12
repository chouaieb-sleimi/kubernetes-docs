# K8S Helm Chart

tags: #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Helm Chart](#k8s-helm-chart)
  - [Chart.yaml](#chartyaml)
  - [Helm Templates](#helm-templates)
  - [Helm Variables](#helm-variables)

<!-- /code_chunk_output -->

---

**Helm Charts** = **Helm Variables** + **Helm Templates** + **Chart.yaml** (chart meta infos)

**Helm Chart Repos:**

- ArtifactHub
  https://artifacthub.io
- Bitnami
  https://charts.bitnami.com/

install helm chart (download, extract, install locally)

```bash
# release-name: chart installation
helm install [release-name] [chart-name]

helm install release-1 bitnami/wordpress
helm install release-2 bitnami/wordpress
helm install release-3 bitnami/wordpress
```

manage helm releases and charts

```bash
helm list
helm uninstall <release-name>
helm pull --untar bitnami/wordpress
```

## Chart.yaml

[[chart_yaml]]

## Helm Templates

[[template]]

## Helm Variables

[[variables]]
