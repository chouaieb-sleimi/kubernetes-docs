# K8S Dynamic AdmissionController

tags: #objects #security

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Overview](#overview)
- [Custom Dynamic Controller Deployment](#custom-dynamic-controller-deployment)

<!-- /code_chunk_output -->

---

## Overview

- In addition to compiled-in admission controller plugins
- developed-as-extensions admission controller;
- run as webhooks configured at runtime

**Admission webhooks**
HTTP callbacks that receive admission requests and do something with them.
You can define two types of admission webhooks:

- **mutating admission webhook**
  modify objects sent to the API server to enforce custom defaults
- **validating admission webhook**
  can reject requests to enforce custom policies

**Control order:**
`mutating admission webhooks` -> `kube-apiserver validation` -> `validating admission webhooks`

## Custom Dynamic Controller Deployment

**1. deploy admission webhook server**

_sample admission server_
https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go

sample deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webhook-server
  namespace: webhook-demo
  labels:
    app: webhook-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-server
  template:
    metadata:
      labels:
        app: webhook-server
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1234
      containers:
        - name: server
          image: stackrox/admission-controller-webhook-demo:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              name: webhook-api
          volumeMounts:
            - name: webhook-tls-certs
              mountPath: /run/secrets/tls
              readOnly: true
      volumes:
        - name: webhook-tls-certs
          secret:
            secretName: webhook-server-tls
```

> Note: this webhook deployment:
> - Denies all request for pod to run as root in container if no securityContext is provided.
> - If no value is set for runAsNonRoot, a default of true is applied, and the user ID defaults to 1234
> - Allow to run containers as root if runAsNonRoot set explicitly to false in the securityContext

**2. create webhook service**

sample service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webhook-server
  namespace: webhook-demo
spec:
  selector:
    app: webhook-server
  ports:
    - port: 443
      targetPort: webhook-api
```

**3. configure webhook on k8s**

- **validating webhooks:** [[validatingWebhookConfiguration]]
- **mutating webhooks:** [[mutatingWebhookConfiguration]]

pod samples (to test mutating admission controller above)

```yaml
# A pod with no securityContext specified.
# Without the webhook, it would run as user root (0). The webhook mutates it
# to run as the non-root user with uid 1234.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-defaults
  labels:
    app: pod-with-defaults
spec:
  restartPolicy: OnFailure
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]

---
# A pod with a securityContext explicitly allowing it to run as root.
# The effect of deploying this with and without the webhook is the same. The
# explicit setting however prevents the webhook from applying more secure
# defaults.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-override
  labels:
    app: pod-with-override
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: false
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]

---
# A pod with a conflicting securityContext setting: it has to run as a non-root
# user, but we explicitly request a user id of 0 (root).
# Without the webhook, the pod could be created, but would be unable to launch
# due to an unenforceable security context leading to it being stuck in a
# 'CreateContainerConfigError' status. With the webhook, the creation of
# the pod is outright rejected.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-conflict
  labels:
    app: pod-with-conflict
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: true
    runAsUser: 0
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]test admission controller pod

kubectl logs <pod-name>
kubectl get po <pod-name> -o yaml | grep -iA3 securitycontext

kubectl create -f /root/pod-with-conflict.yaml
  Error from server: error when creating "/root/pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)

```
