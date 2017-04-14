# Create Namespaces

Namespaces serve as a logical division of resources within a Kubernetes cluster.
When combined with RBAC, it becomes possible to enforce the divisions.

These two Kubernetes namespaces will be used for showing how to grant different
access to different users in different parts of the cluster.

```sh
a-kubectl create namespace ns-1
a-kubectl create namespace ns-2
```

