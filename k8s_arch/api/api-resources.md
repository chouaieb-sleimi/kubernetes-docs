# K8S Resources

        KIND                            NAME                              SHORTNAMES   NAMESPACED
        Pod                             pods                              po           true
        ReplicationController           replicationcontrollers            rc           true
        ReplicaSet                      replicasets                       rs           true
        Deployment                      deployments                       deploy       true
        StatefulSet                     statefulsets                      sts          true
        LimitRange                      limitranges                       limits       true
        ConfigMap                       configmaps                        cm           true
        Secret                          secrets                                        true
        Job                             jobs                                           true
        CronJob                         cronjobs                          cj           true

        Service                         services                          svc          true
        ServiceAccount                  serviceaccounts                   sa           true
        Role                            roles                                          true
        RoleBinding                     rolebindings                                   true
        Ingress                         ingresses                         ing          true
        NetworkPolicy                   networkpolicies                   netpol       true
        ClusterRole                     clusterroles                                   false
        ClusterRoleBinding              clusterrolebindings                            false

        PersistentVolumeClaim           persistentvolumeclaims            pvc          true
        Node                            nodes                             no           false
        Namespace                       namespaces                        ns           false
        PersistentVolume                persistentvolumes                 pv           false
        StorageClass                    storageclasses                    sc           false

        HelmChartConfig                 helmchartconfigs                               true
        HelmChart                       helmcharts                                     true
        CustomResourceDefinition        customresourcedefinitions         crd,crds     false
        MutatingWebhookConfiguration    mutatingwebhookconfigurations                  false
        ValidatingWebhookConfiguration  validatingwebhookconfigurations                false
