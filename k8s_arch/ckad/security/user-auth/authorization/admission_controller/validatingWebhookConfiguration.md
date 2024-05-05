# K8S ValidatingWebhookConfiguration

tags: #objects #security

---

validatingWebhookConfiguration sample

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
  - name: "pod-policy.example.com"
    clientConfig:
      # if webhook server deployed outside the cluster
      #url: <webhook-server-url>
      service:
        namespace: "webhook-namespace"
        name: "webhook-service"
      caBundle: <CA_BUNDLE> #to comm w/ the webhook server
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE"]
        resources: ["pods"]
        scope: "Namespaced"
    admissionReviewVersions: ["v1"]
    sideEffects: None
    timeoutSeconds: 5
```
