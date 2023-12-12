## K8S Containers Commands And Arguments

tags: #objects #configuration #workloads

---

define commands and arguments to [[containers]]

`IMAGE/entrypoint = K8S/command` and `IMAGE/cmd = K8S/args`

Containerfile

    ENTRYPOINT ["python", "app.py"]
    CMD ["--color", "red"]

equivalent in **Pod YAML**

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["python", "app.py"]
        args: ["--color", "red"]
