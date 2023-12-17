# K8S Helm Chart Variables

tags: #helm #workloads

---

define helm variables

```bash
cat values.yaml
  # image: wordpress:4.8-apache
  # storage: 20Gi
  # passwordEncoded: DkfEhMs.....
```

use helm variables in a `values.yaml`

```bash
cat templates/pv.yaml
  ...
  {{ .Values.storage }}
  ...
```
