# Using Existing Cluster Roles

To divide a cluster between multiple teams, there are [existing cluster
roles](https://kubernetes.io/docs/admin/authorization/rbac/#user-facing-roles)
that are pre-configured to help with that. These are the steps to take use those
cluster roles to configure users with admin priviledges in their own namespaces,
and viewing permissions in the other namespace.

The pre-configured cluster role to grant admin priviledge to a namespace is the
`admin` cluster role.

```sh
a-kubectl get clusterroles
```

Make the users administrators in their own namespaces by binding the cluster
role to the user in the namespace where they should be administrators.

```sh
a-kubectl create rolebinding account1-admin \
    --clusterrole=admin \
    --user=$account1  \
    --namespace=ns-1
a-kubectl create rolebinding account2-admin \
    --clusterrole=admin \
    --user=$account2  \
    --namespace=ns-2
```

To make the users viewers in the opposite namespace, bind the `viewer` cluster
role in the namespace where they should be a viewer.

```sh
a-kubectl create rolebinding account1-view \
    --clusterrole=view \
    --user=$account1 \
    --namespace=ns-2
a-kubectl create rolebinding account2-view \
    --clusterrole=view \
    --user=$account2 \
    --namespace=ns-1
```

