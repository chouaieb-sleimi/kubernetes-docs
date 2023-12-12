# K8S User Authentication

tags: #security

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S User Authentication](#k8s-user-authentication)
  - [Configure Users - Server](#configure-users---server)
    - [Static Password File](#static-password-file)
    - [Static Token File](#static-token-file)
    - [Certificates](#certificates)
  - [Configure Users - Client](#configure-users---client)

<!-- /code_chunk_output -->

---

Types of accounts:

- Service ccounts (bots and machines)
- User accounts
  **not managed by k8s**, it is **handled externally** via:
  - static password file\* (Deprecated in 1.19)
  - static token file\* (Deprecated in 1.19)
  - certificates
  - identity service (ex: LDAP)

> Notes:
> - file auth not recommended
> - consider using volume mount while providing the auth file in a kubeadm setup
> - setup RBAC for new users

## Configure Users - Server

### Static Password File

static password file sample

    cat user-details.csv

    # password,usernae,uid,group (optional)
    password123,user1,u0001,group1
    password123,user2,u0002,group2
    password123,user3,u0003,group3
    password123,user4,u0004,group4

use static password file in `kube-apiserver`

    /usr/local/bin/kube-apiserver \
      ...
      --base-file-auth=user-details.csv
      ...

use account auth in API call

    curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"

### Static Token File

static token file sample

    cat user-token-details.csv

    # password,usernae,uid,group (optional)
    b026324c6904b2a9cb4b88d6d61c81d1,user1,u0001,group1
    26ab0db90d72e28ad0ba1e22ee510510,user2,u0002,group2
    6d7fce9fee471194aa8b5b6e47267f03,user3,u0003,group3
    48a24b70a0b376535542b996af517398,user4,u0004,group4

use static token file in `kube-apiserver`

    /usr/local/bin/kube-apiserver \
      ...
      --token-auth-file=user-token-details.csv
      ...

use account auth in API call

    curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer b026324c6904b2a9cb4b88d6d61c81d1"

### Certificates

use cert auth in API call

    curl https://my-kube-playground:6443/api/v1/pods \
    --key admin.key \
    --cert admin.crt \
    --cacert ca.crt

use cert auth in kubectl

    kubectl get pods \
      --server my-kube-playground:6443 \
      --client-key admin.key \
      --client-certificate admin.crt \
      --certificate-authority ca.crt

## Configure Users - Client

[[kube_config]]
