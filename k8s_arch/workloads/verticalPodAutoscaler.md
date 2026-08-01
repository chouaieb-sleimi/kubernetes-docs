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
      kind: Deployment
      name: my-app
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
- `kubectl describe vpa flask-app`
  ```
  Name:         flask-app
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  autoscaling.k8s.io/v1
  Kind:         VerticalPodAutoscaler
  Metadata:
    Creation Timestamp:  2026-04-30T19:15:56Z
    Generation:          1
    Resource Version:    8279
    UID:                 5df78000-af08-4da4-aa2b-46bb36d1c9ce
  Spec:
    Resource Policy:
      Container Policies:
        Container Name:  *
        Controlled Resources:
          cpu
          memory
        Max Allowed:
          Cpu:     1
          Memory:  500Mi
        Min Allowed:
          Cpu:     100m
          Memory:  100Mi
    Target Ref:
      API Version:  apps/v1
      Kind:         Deployment
      Name:         flask-app
    Update Policy:
      Eviction Requirements:
        Change Requirement:  TargetHigherThanRequests
        Resources:
          cpu
          memory
      Update Mode:  Recreate
  Status:
    Conditions:
      Last Transition Time:  2026-04-30T19:16:20Z
      Status:                True
      Type:                  RecommendationProvided
    Recommendation:
      Container Recommendations:
        Container Name:  flask-app
        Lower Bound:
          Cpu:     100m
          Memory:  250Mi
        Target:
          Cpu:     100m
          Memory:  250Mi
        Uncapped Target:
          Cpu:     25m
          Memory:  250Mi
        Upper Bound:
          Cpu:     100m
          Memory:  250Mi
  Events:
    Type    Reason      Age   From         Message
    ----    ------      ----  ----         -------
    Normal  EvictedPod  2s    vpa-updater  VPA Updater evicted Pod flask-app-8676bb879b-xk9z5 to apply resource recommendation.
  ```
