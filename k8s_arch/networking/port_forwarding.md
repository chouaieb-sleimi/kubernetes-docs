### K8S Port Forwarding

tags: #network

---

Environment

```bash
kubectl get service mongo
# NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
# mongo   ClusterIP   10.96.41.183   <none>        27017/TCP   11s

kubectl get pods
# NAME                     READY   STATUS    RESTARTS   AGE
# mongo-75f59d57f4-4nd6q   1/1     Running   0          2m4s

kubectl get pod mongo-75f59d57f4-4nd6q --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
# 27017
```

> Note: `27017` is the TCP port allocated to `mongo` pod_

Forward a local port to a port on the Pod

```bash
kubectl port-forward  mongo-75f59d57f4-4nd6q       28015:27017
                      pods mongo-75f59d57f4-4nd6q  28015:27017
                      deployment mongo             28015:27017
                      replicaset mongo-75f59d57f4  28015:27017
                      service mongo                28015:27017

# Forwarding from 127.0.0.1:28015 -> 27017
# Forwarding from [::1]:28015 -> 27017
```

let kubectl choose the local port

```bash
kubectl port-forward deployment/mongo :27017
# Forwarding from 127.0.0.1:63753 -> 27017
# Forwarding from [::1]:63753 -> 27017
```
