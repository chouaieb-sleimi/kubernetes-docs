# Table of Contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Table of Contents](#table-of-contents)
- [Helm Resources](#helm-resources)
- [Helm Notes](#helm-notes)
  - [YAML Syntax](#yaml-syntax)
    - [Scalars and Collections](#scalars-and-collections)
    - [Strings in YAML](#strings-in-yaml)
    - [Embedding Multiple Documents in One File](#embedding-multiple-documents-in-one-file)
    - [YAML is a Superset of JSON](#yaml-is-a-superset-of-json)
    - [YAML Anchors](#yaml-anchors)
  - [Built-in Objects](#built-in-objects)
  - [Values Files](#values-files)
  - [Functions and Pipelines](#functions-and-pipelines)
  - [Flow Control](#flow-control)
    - [`if/else` conditional blocks](#ifelse-conditional-blocks)
    - [`with` scope definitions](#with-scope-definitions)
    - [`range` Looping Action](#range-looping-action)
    - [Controlling Whitespace `{{-` `-}}`](#controlling-whitespace----)
  - [Variables](#variables)
  - [Named Templates](#named-templates)
    - [Partials and \_ files](#partials-and-_-files)
    - [`define` and `template` Actions](#define-and-template-actions)
    - [`include` Function](#include-function)
  - [Accessing Files Inside Templates](#accessing-files-inside-templates)
    - [Path helpers](#path-helpers)
    - [Glob patterns](#glob-patterns)
    - [ConfigMap and Secrets utility functions](#configmap-and-secrets-utility-functions)
    - [Encoding](#encoding)
    - [Lines](#lines)
  - [Sub-Charts and Global Values](#sub-charts-and-global-values)
  - [`.helmignore` File](#helmignore-file)
  - [Debugging Templates](#debugging-templates)

<!-- /code_chunk_output -->

---

# Helm Resources

- **Helm docs**
  https://helm.sh
- **Charts Workflow**
  https://helm.sh/docs/topics/charts/
- **Go template docs - template syntax**
  https://godoc.org/text/template
- **Helm Charts Tips and Tricks**
  https://helm.sh/docs/howto/charts_tips_and_tricks/
- **Helm Chart Hooks Guide - lifecycle hooks**
  https://helm.sh/docs/topics/charts_hooks/
- **Sprig: more than sixty of the template functions.**
  https://github.com/Masterminds/sprig
- **Schelm tool: debugging charts**
  https://github.com/databus23/schelm
- **CNCF Artifact Hub charts repo**
  https://artifacthub.io/packages/search?kind=0
- **K8S resources**
  https://kubernetes.io/docs/home/

---

# Helm Notes

---

## YAML Syntax

- YAML format
  https://helm.sh/docs/chart_template_guide/yaml_techniques/

### Scalars and Collections

- **collection types:**

  - **maps**
  - **sequences**

```yaml
map:
  one: 1
  two: 2
  three: 3

sequence:
  - one
  - two
  - three
```

- **scalar types:** (individual values as opposed to collections)

```yaml
count: 1 # int
size: 2.34 # float
---
count: "1" # <-- string, not int
size: "2.34" # <-- string, not float
---
isGood: true # bool
answer: "true" # string
```

- `!!str` tells parser that `age` is a string, even if it looks like an int
- `port` is treated as an int, even though it is quoted

```yaml
coffee: "yes, please"
age: !!str 21
port: !!int "80"
```

### Strings in YAML

multi-line strings

```yaml
# coffee:'Latte\nCappuccino\nEspresso\n'

coffee: |
  Latte
  Cappuccino
  Espresso
```

controlling spaces in multi-line strings

```yaml
# strip off the trailing newline
# coffee: 'Latte\nCappuccino\nEspresso'
coffee: |-
  Latte
  Cappuccino
  Espresso

---
# preserve all trailing whitespace
# coffee: 'Latte\nCappuccino\nEspresso\n\n\n'
coffee: |+
  Latte
  Cappuccino
  Espresso  


another: value
---
# preserve indentation inside text block
# coffee: 'Latte\n 12 oz\n 16 oz\nCappuccino\nEspresso'
coffee: |-
  Latte
    12 oz
    16 oz
  Cappuccino
  Espresso
```

folded multi-line strings

```yaml
# declare a folded block
# coffee: 'Latte Cappuccino Espresso\n'
coffee: >
  Latte
  Cappuccino
  Espresso


# trim all newlines
# coffee: 'Latte\n 12 oz\n 16 oz\nCappuccino Espresso'
coffee: >-
  Latte
    12 oz
    16 oz
  Cappuccino
  Espresso
```

### Embedding Multiple Documents in One File

- some files in Helm cannot contain more than one doc
  - if more than one document is provided in `values.yaml` file, only the first will be used.
- template files w/ more than one document is treated as one object during template rendering
  - resulting YAML is split into multiple documents before it is fed to k8s

```yaml
---
document:1
---
document: 2
```

### YAML is a Superset of JSON

- files such as `values.yaml` may contain JSON data
  - Helm does not treat the file extension `.json` as a valid suffix.

```json
// JSON representation
{
  "coffee": "yes, please",
  "coffees": ["Latte", "Cappuccino", "Espresso"]
}
```

```yaml
# YAML representation
coffees:
  - Latte
  - Cappuccino
  - Espresso
```

```yaml
# YAML and JSON can be mixed (with care)
coffee: "yes, please"
coffees: ["Latte", "Cappuccino", "Espresso"]
```

### YAML Anchors

- a way to store a reference to a value, and later refer to that value by reference
- first time the YAML is consumed, the reference is expanded and then discarded.

```yaml
coffee: "yes, please"
favorite: &favoriteCoffee "Cappuccino"
coffees:
  - Latte
  - *favoriteCoffee
  - Espresso
---
# resulting manifest
coffee: yes, please
favorite: Cappuccino
coffees:
  - Latte
  - Cappuccino
  - Espresso
```

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
  - `Template.BasePath`
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

---

## Values Files

- see:
  https://helm.sh/docs/chart_template_guide/values_files/

---

## Functions and Pipelines

- see:
  https://helm.sh/docs/chart_template_guide/functions_and_pipelines/

---

## Flow Control

https://helm.sh/docs/chart_template_guide/control_structures/

flow controls:

### `if/else` conditional blocks
### `with` scope definitions
### `range` Looping Action
### Controlling Whitespace `{{-` `-}}`

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

---

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

---

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

---

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

### ConfigMap and Secrets utility functions

> Note: available **after version 2.0.2**

```yaml
# using folder structure in previous section:
apiVersion: v1
kind: ConfigMap
metadata:
  name: conf
data:
{{ (.Files.Glob "foo/*").AsConfig | indent 2 }}

---
apiVersion: v1
kind: Secret
metadata:
  name: very-secret
type: Opaque
data:
{{ (.Files.Glob "bar/*").AsSecrets | indent 2 }}
```

### Encoding

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
data:
  token: |-
        {{ .Files.Get "config1.toml" | b64enc }}

---
# Source: mychart/templates/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: lucky-turkey-secret
type: Opaque
data:
  token: |-
        bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
```

### Lines

used to access each line of a file in template

```yaml
data:
  some-file.txt: {{ range .Files.Lines "foo/bar.txt" }}
    {{ . }}{{ end }}
```

---

## Sub-Charts and Global Values

---

## `.helmignore` File

differences from `.gitignore`:

- `**` syntax is not supported.
- globbing library is Go's `filepath.Match`, not `fnmatch(3)`
- trailing spaces are always ignored (there is no supported escape sequence)
- no support for `!` as a special leading sequence.
- does not exclude itself by default, you have to add an explicit entry for `.helmignore`

`.helmignore` example

```bash
# comment

# Match any file or path named .helmignore
.helmignore

# Match any file or path named .git
.git

# Match any text file
*.txt

# Match only directories named mydir
mydir/

# Match only text files in the top-level directory
/*.txt

# Match only the file foo.txt in the top-level directory
/foo.txt

# Match any file named ab.txt, ac.txt, or ad.txt
a[b-d].txt

# Match any file under subdir matching temp*
*/temp*

*/*/temp*
temp?
```

---

## Debugging Templates

```yaml
# debug templates
# verify chart follows best practices
helm lint

# test render templates locally
helm template --debug

# render templates, then return resulting manifest files
helm install --dry-run --debug

# see what templates are installed on server
helm get manifest RELEASE_NAME
```

skip YAML parse errors blockings

```yaml
apiVersion: v2
# some: problem section
# {{ .Values.foo | quote }}
The above will be rendered and returned with the comments intact:

apiVersion: v2
# some: problem section
#  "bar"
```
