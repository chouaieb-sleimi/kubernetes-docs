## K8S Containers Commands And Arguments

tags: #configuration #workloads

---

- define commands and arguments to [[containers]]
- `Dockerfile/entrypoint` = `pod.container/command`
- `Dockerfile/cmd` = `pod.container/args`
  - `pod.container` takes precedence over `Dockerfile`
Containerfile

    ...
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
