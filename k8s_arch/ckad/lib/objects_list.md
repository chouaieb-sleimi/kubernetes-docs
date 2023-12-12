# CKAD Objects List

tags: #api 

---

`kubectl api-resources`

| NAME                            | SHORTNAMES | APIVERSION                      | NAMESPACED | KIND                           |
| ------------------------------- | ---------- | ------------------------------- | ---------- | ------------------------------ |
| namespaces                      | ns         | v1                              | false      | Namespace                      |
| nodes                           | no         | v1                              | false      | Node                           |
| pods                            | po         | v1                              | true       | Pod                            |
| replicationcontrollers          | rc         | v1                              | true       | ReplicationController          |
| limitranges                     | limits     | v1                              | true       | LimitRange                     |
| resourcequotas                  | quota      | v1                              | true       | ResourceQuota                  |
| configmaps                      | cm         | v1                              | true       | ConfigMap                      |
| secrets                         |            | v1                              | true       | Secret                         |
| services                        | svc        | v1                              | true       | Service                        |
| serviceaccounts                 | sa         | v1                              | true       | ServiceAccount                 |
| persistentvolumes               | pv         | v1                              | false      | PersistentVolume               |
| persistentvolumeclaims          | pvc        | v1                              | true       | PersistentVolumeClaim          |
|                                 |            |                                 |            |                                |
| storageclasses                  | sc         | storage.k8s.io/v1               | false      | StorageClass                   |
|                                 |            |                                 |            |                                |
| mutatingwebhookconfigurations   |            | admissionregistration.k8s.io/v1 | false      | MutatingWebhookConfiguration   |
| validatingwebhookconfigurations |            | admissionregistration.k8s.io/v1 | false      | ValidatingWebhookConfiguration |
| customresourcedefinitions       | crd,crds   | apiextensions.k8s.io/v1         | false      | CustomResourceDefinition       |
|                                 |            |                                 |            |                                |
| deployments                     | deploy     | apps/v1                         | true       | Deployment                     |
| replicasets                     | rs         | apps/v1                         | true       | ReplicaSet                     |
| statefulsets                    | sts        | apps/v1                         | true       | StatefulSet                    |
| jobs                            |            | batch/v1                        | true       | Job                            |
| cronjobs                        | cj         | batch/v1                        | true       | CronJob                        |
|                                 |            |                                 |            |                                |
| clusterroles                    |            | rbac.authorization.k8s.io/v1    | false      | ClusterRole                    |
| clusterrolebindings             |            | rbac.authorization.k8s.io/v1    | false      | ClusterRoleBinding             |
| roles                           |            | rbac.authorization.k8s.io/v1    | true       | Role                           |
| rolebindings                    |            | rbac.authorization.k8s.io/v1    | true       | RoleBinding                    |
|                                 |            |                                 |            |                                |
| ingresses                       | ing        | networking.k8s.io/v1            | true       | Ingress                        |
| networkpolicies                 | netpol     | networking.k8s.io/v1            | true       | NetworkPolicy                  |
|                                 |            |                                 |            |                                |
| helmchartconfigs                |            | helm.cattle.io/v1               | true       | HelmChartConfig                |
| helmcharts                      |            | helm.cattle.io/v1               | true       | HelmChart                      |
