# K8S Ingress

tags: #object #network #security

---

demo/lab image: kodekloud/webapp-conntest

- port-based or IP-based trafic control
- specify how a pod is allowed to communicate with network "entities"
- applies to connections to and from a pod

Network solutions that **support network policy:**

- Kube-router
- Calico
- Romana
- Weave-net

Network solutions that **do not support network policy:**

- Flannel
  if network policy is created, no error message will be displayed, the net policy will simply not work

network policy sample

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default

spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        # And applied here because both rules
        # are in the same from element
        - namespaceSelector:
            matchLabels:
              env: prod
          podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```
