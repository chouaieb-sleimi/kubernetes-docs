# K8S CustomResourceDefinition

tags: #objects #arch

---

Represents a custom current/desired state of k8s resource. It is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation.

CRD/custom resource sample 1

```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: myflight-ticket
spec:
  from: Mumbai
  to: London
  number: 2

---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com

spec:
  scope: Namespaced
  # api group
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft

  versions:
    - name: v1
      served: true # preferred version
      storage: true # storage version

  schema:
    openAPIV3Schema:
      type: Object
      properties:
        spec:
          type: Object
          properties:
            from:
              type: string
            to:
              type: string
            number:
              type: integer
              minimum: 1
              maximum: 10
```

CRD/custom resource sample 2

```yaml
apiVersion: traffic.controller/v1
kind: Global
metadata:
  name: datacenter
spec:
  dataField: 2
  access: true

---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: globals.traffic.controller
spec:
  conversion:
    strategy: None
  group: traffic.controller
  names:
    kind: Global
    listKind: GlobalList
    plural: globals
    shortNames:
      - gb
    singular: global
  scope: Namespaced
  versions:
    - name: v1
      schema:
        openAPIV3Schema:
          properties:
            spec:
              properties:
                access:
                  type: boolean
                dataField:
                  type: integer
              type: object
          type: object
      served: true
      storage: true
```
