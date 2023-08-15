# CKAD Tips

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [CKAD Tips](#ckad-tips)
  - [Student Tips](#student-tips)
  - [Certification Tip: Imperative Commands](#certification-tip-imperative-commands)
    - [POD](#pod)
    - [Deployment](#deployment)
    - [Service](#service)
  - [A quick note on editing Pods and Deployments](#a-quick-note-on-editing-pods-and-deployments)
    - [Edit a POD](#edit-a-pod)
    - [Edit Deployments](#edit-deployments)
  - [FAQ - What is the rewrite-target option?](#faq---what-is-the-rewrite-target-option)

<!-- /code_chunk_output -->

## Student Tips

Make sure you check out these tips and tricks from other students who have cleared the exam:

https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/

https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014

https://github.com/lucassha/CKAD-resources

## Certification Tip: Imperative Commands

While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one-time tasks done quickly, as well as generate a definition template easily. This would help save a considerable amount of time during your exams.

Before we begin, familiarize yourself with the two options that can come in handy while working with the below commands:

`--dry-run`: By default, as soon as the command is run, the resource will be created. If you simply want to test your command, use the `--dry-run=client` option. This will not create the resource. Instead, tell you whether the resource can be created and if your command is right.

`-o yaml`: This will output the resource definition in YAML format on the screen.

Use the above two in combination along with Linux output redirection to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

    kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml

### POD

Create an NGINX Pod

    kubectl run nginx --image=nginx

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

    kubectl run nginx --image=nginx --dry-run=client -o yaml

### Deployment

Create a deployment

    kubectl create deployment --image=nginx nginx

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

    kubectl create deployment --image=nginx nginx --dry-run -o yaml

Generate Deployment with 4 Replicas

    kubectl create deployment nginx --image=nginx --replicas=4

You can also scale deployment using the kubectl scale command.

    kubectl scale deployment nginx --replicas=4

Another way to do this is to save the YAML definition to a file and modify

    kubectl create deployment nginx --image=nginx--dry-run=client -o yaml > nginx-deployment.yaml

You can then update the YAML file with the replicas or any other field before creating the deployment.

### Service

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

    kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

(This will automatically use the pod's labels as selectors)

Or

    kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml

_This will not use the pods' labels as selectors; instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work well if your pod has a different label set. So generate the file and modify the selectors before creating the service_

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

    kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml

_This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod._

Or

    kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml

_This will not use the pods' labels as selectors_

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

Reference:

https://kubernetes.io/docs/reference/kubectl/conventions/

## A quick note on editing Pods and Deployments

### Edit a POD

Remember, you CANNOT edit specifications of an existing POD other than the below.

    spec.containers[*].image
    spec.initContainers[*].image
    spec.activeDeadlineSeconds
    spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod <pod name>` command. This will **open the pod specification in an editor** (vi editor). Then **edit the required properties.** When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

`kubectl delete pod webapp`

Then create a new pod with your changes using the temporary file

`kubectl create -f /tmp/kubectl-edit-ccvrq.yaml`

2. The second option is to **extract the pod definition in YAML format to a file** using the command

`kubectl get pod webapp -o yaml > my-new-pod.yaml`

Then make the changes to the exported file using an editor (vi editor). Save the changes

`vi my-new-pod.yaml`

Then delete the existing pod

`kubectl delete pod webapp`

Then create a new pod with the edited file

`kubectl create -f my-new-pod.yaml`

### Edit Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification, with every change **the deployment will automatically delete and create a new pod with the new changes.** So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

`kubectl edit deployment my-deployment`

## FAQ - What is the rewrite-target option?

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.

Our `watch` app displays the video streaming webpage at `http://<watch-service>:<port>/`

Our `wear` app displays the apparel webpage at `http://<wear-service>:<port>/`

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

    http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/

    http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/

Without the `rewrite-target` option, this is what would happen:

    http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch

    http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear

Notice `watch` and `wear` at the end of the target URLs. The target applications are not configured with `/watch` or `/wear` paths. They are different applications built specifically for their purpose, so they don't expect `/watch` or `/wear` in the URLs. And as such the requests would fail and throw a `404` not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the `rewrite-target` option. This rewrites the URL by replacing whatever is under `rules->http->paths->path` which happens to be `/pay` in this case with the value in `rewrite-target`. This works just like a search and replace function.

For example: `replace(path, rewrite-target)`
In our case: `replace("/path","/")`

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: test-ingress
      namespace: critical-space
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /pay
            backend:
              service:
                name: pay-service
                port: 8282

In another example given [here](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), this could also be:

`replace("/something(/|$)(.*)", "/$2")`

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$2
      name: rewrite
      namespace: default
    spec:
      rules:
      - host: rewrite.bar.com
        http:
          paths:
          - backend:
              service:
                name: http-svc
                port: 80
            path: /something(/|$)(.*)
