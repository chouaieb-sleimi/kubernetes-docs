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
  - [Accessing Files Inside Templates](#accessing-files-inside-templates)
    - [Path helpers](#path-helpers)
    - [Glob patterns](#glob-patterns)

<!-- /code_chunk_output -->

---

## Built-in Objects

https://helm.sh/docs/chart_template_guide/builtin_objects/

- **`Values`**
  variables from `values.yaml` file and from user-supplied files
- **`Chart`**
  contents of the `Chart.yaml` file
- **`Release`**
  describes the release itself
  - `Release.Name`
  - `Release.Namespace`
  - `Release.IsUpgrade`
  - `Release.IsInstall`
  - `Release.Revision`
  - `Release.Service`
- **`Template`**
  information about the current template that is being executed`
  - `Template.Name`
  - `Template.BasePath
- **`Files`**
  provides access to all non-special files in a chart
  - `Files.Get`
  - `Files.GetBytes`
  - `Files.Glob`
  - `Files.Lines`
  - `Files.AsSecrets`
  - `Files.AsConfig`
- **`Capabilities`**
  provides information about what capabilities the k8s cluster supports
  - `Capabilities.APIVersions`
  - `Capabilities.APIVersions.Has`
  - `Capabilities.KubeVersion`
  - `Capabilities.KubeVersion.Major`
  - `Capabilities.KubeVersion.Minor`
  - `Capabilities.HelmVersion`
  - `Capabilities.HelmVersion.Version`
  - `Capabilities.HelmVersion.GitCommit`
  - `Capabilities.HelmVersion.GitTreeState`
  - `Capabilities.HelmVersion.GoVersion`

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

## Accessing Files Inside Templates

> Note:
>
> - Charts must be smaller than 1M due to storage limitations of k8s objects
> - file-level permissions will have no impact on the availability of a file when it comes to the `.Files` object.
> - some files cannot be accessed through `.Files` object for security reasons:
>   - files in `templates/` cannot be accessed.
>   - files excluded using `.helmignore` cannot be accessed.
>   - files outside of a helm application `subchart`, including those of the parent, cannot be accessed

```yaml

# config1.toml:
# message = Hello from config 1
#
# config2.toml:
# message = This is config 2
#
# config3.toml:
# message = Goodbye from config 3

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $files := .Files }}
  {{- range tuple "config1.toml" "config2.toml" "config3.toml" }}
  {{ . }}: |-
        {{ $files.Get . }}
  {{- end }}

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: quieting-giraf-configmap
data:
  config1.toml: |-
        message = Hello from config 1

  config2.toml: |-
        message = This is config 2

  config3.toml: |-
        message = Goodbye from config 3
```

### Path helpers

functions from Go's `path` package are all accessible with the same names as in the Go package (`Base` becomes `base`, etc.)

The imported functions are:

- Base
- Dir
- Ext
- IsAbs
- Clean

### Glob patterns

- GOPlang docs: glob patterns
  https://pkg.go.dev/github.com/gobwas/glob

```yaml
#foo/:
#  foo.txt foo.yaml
#
#bar/:
#  bar.go bar.conf baz.yaml

---
# option 1
{{ $currentScope := .}}
{{ range $path, $_ :=  .Files.Glob  "**.yaml" }}
    {{- with $currentScope}}
        {{ .Files.Get $path }}
    {{- end }}
{{ end }}


---
# option 2
{{ range $path, $_ :=  .Files.Glob  "**.yaml" }}
      {{ $.Files.Get $path }}
{{ end }}
```
