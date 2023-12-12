# Helm Notes

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Helm Notes](#helm-notes)
  - [Built-in Objects](#built-in-objects)
  - [Flow Control](#flow-control)
  - [Variables](#variables)
  - [Named Templates](#named-templates)
    - [Partials and \_ files](#partials-and-_-files)
    - [`define` and `template` Actions](#define-and-template-actions)
    - [`include` Function](#include-function)

<!-- /code_chunk_output -->

---

## Built-in Objects

https://helm.sh/docs/chart_template_guide/builtin_objects/

- **Chart**
- **Values**
- **Release**
- **Files**
- **Capabilities**
- **Template**

## Flow Control

https://helm.sh/docs/chart_template_guide/control_structures/

flow controls:

- `if/else` conditional blocks
- `with` scope definitions
- `range` Looping Action

```yaml
# values.yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee" }}
  mug: "true"
  {{- end }}
  toppings: |-
    {{- range $.Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}
  {{- end }}

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: edgy-dragonfly-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: "true"
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"
```

**Notes**

- `$` is mapped to the root scope
- `toppings: |-` line is declaring a multi-line string
  the list of toppings is not a YAML list. It's a big string

## Variables

https://helm.sh/docs/chart_template_guide/variables/

- variables can be accessed without respect to the present scope

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```

```yaml
toppings: |-
  {{- range $index, $topping := .Values.pizzaToppings }}
    {{ $index }}: {{ $topping }}
  {{- end }}
```

```yaml
toppings: |-
  0: mushrooms
  1: cheese
  2: peppers
  3: onions
```

## Named Templates

https://helm.sh/docs/chart_template_guide/named_templates/

### Partials and \_ files

### `define` and `template` Actions

`define`: create a named template inside of a template file (does not produce output unless it is called with a `template`)
`template`: includes created templates

```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: running-panda-configmap
  labels:
    generator: helm
    date: 2016-11-02
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
```

**Note**
if we call template w/ `{{- template "mychart.labels" }}` the template `mychart.labels` will have no access to objects in the scope `.`

### `include` Function

`template`: action
`include`: function

- can be passed through pipelines
- preferable to use `include` over `template` in Helm templates

```yaml
{{- define "mychart.app" -}}
app_name: {{ .Chart.Name }}
app_version: "{{ .Chart.Version }}"
{{- end -}}

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
{{ include "mychart.app" . | indent 4 }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
{{ include "mychart.app" . | indent 2 }}

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: edgy-mole-configmap
  labels:
    app_name: mychart
    app_version: "0.1.0"
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
  app_name: mychart
  app_version: "0.1.0"
```
