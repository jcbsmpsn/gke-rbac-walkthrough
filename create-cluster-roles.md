# Creating Cluster Roles

Additional permissions will be granted to *account1* and *account2* for specific
namespaces using a common role.

## Service Account 1 Listing Deployments in Namespace 1

1. Try to list the pods in `ns-1` using GCP Service Account 1.
    ```sh
    1-kubectl get deployments --namespace=ns-1
    ```
2. Why not just create a role without specifying a namespace, so that it isn't
   in a namespace and can be used anywhere?
    ```sh
    a-kubectl create role deployment-reader \
        --verb=get \
        --verb=list \
        --verb=watch \
        --resource=deployments
    ```
    Well, actually, if you don't specify a namespace, it just goes into the
    default namespace, so it's still in a namespace. The default namespace is
    just like all the other namespaces, except it's where things go by default.
    ```sh
    a-kubectl get roles --all-namespaces
    ```
3. Create a cluster role that contains the necessary permissions:
    ```sh
    a-kubectl create clusterrole deployment-reader \
        --verb=get \
        --verb=list \
        --verb=watch \
        --resource=deployments
    ```
4. Create a role binding to associate the role with GCP Service Account 1.
    ```sh
    a-kubectl create rolebinding account1-deployment-reader-binding \
        --clusterrole=deployment-reader \
        --user=$account1 \
        --namespace=ns-1
    ```
5. Try again.
    ```sh
    1-kubectl get deployments --namespace=ns-1
    ```

## Service Account 2 Listing Deployments in Namespace 2

1. Try to list the pods in `ns-2` using GCP Service Account 2.
    ```sh
    2-kubectl get deployments --namespace=ns-2
    ```
2. Re-use the cluster role that was created.
    ```sh
    a-kubectl create rolebinding account2-deployment-reader-binding \
        --clusterrole=deployment-reader \
        --user=$account2 \
        --namespace=ns-2
    ```
3. List the pods in `ns-2` using GCP Service Account 2.
    ```sh
    2-kubectl get deployments --namespace=ns-2
    ```
