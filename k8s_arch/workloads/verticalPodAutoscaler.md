# K8S VerticalPodAutoscaler

tags: #objects #workloads

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->

---

- **automatically adjust** the CPU and memory requests and limits for pods based on historical resource usage.
- must be added as an additional component to the cluster, as it is not included in a default k8s installation.
- **VPA installation:** `kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/vertical-pod-autoscaler/deploy/vpa.yaml`
- consists of **3 main components:**
  - **VerticalPodAutoscalerRecommender (vpa-recommender)**: monitors the resource usage of pods and makes recommendations for resource adjustments.
  - **VerticalPodAutoscalerController (vpa-controller/updater)**: applies the recommendations made by the recommender to the pods.
  - **VerticalPodAutoscalerAdmissionController (vpa-admission-controller)**: intercepts pod creation requests and adjusts the resource requests and limits based on the recommendations.
- example VPA resource definition:
  ```yaml
  apiVersion: autoscaling.k8s.io/v1
  kind: VerticalPodAutoscaler
  metadata:
    name: my-app-vpa
  spec:
    targetRef:
      apiVersion: "apps/v1"
      kind:       Deployment
      name:       my-app
    updatePolicy:
      updateMode: "Auto"
    containerPolicies:
    - containerName: "my-app"
      minAllowed:
        cpu: "100m"
        memory: "200Mi"
      maxAllowed:
        cpu: "1"
        memory: "1Gi"
      controlledResources: ["cpu", "memory"]
  ```
- VPA can operate in three modes:
  - **Off**: VPA only makes recommendations but does not apply them.
  - **Initial**: VPA sets resource requests for new pods based on recommendations but does not update existing pods.
  - **Auto**: VPA automatically updates resource requests for both new and existing pods based on recommendations. (behaves like Recreate mode for now)
  - **Recreate**: VPA evicts pods to apply new resource requests, causing them to be recreated with updated values.
  