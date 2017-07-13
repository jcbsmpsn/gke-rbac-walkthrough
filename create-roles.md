# Creating Roles

Additional permissions will be granted to *account1* and *account2* for specific
namespaces.

## Service Account 1 Listing Pods in Namespace 1

1. Try to list the pods in `ns-1` using GCP Service Account 1.
    ```sh
    1-kubectl get pods --namespace=ns-1
    ```
2. Create a role that contains the necessary permissions.
    ```sh
    a-kubectl create role pod-reader \
        --verb=get \
        --verb=list \
        --verb=watch \
        --resource=pods \
        --namespace=ns-1
    ```

    Note: If you get an error at this point:

    ```sh
    Error from server (Forbidden): roles.rbac.authorization.k8s.io "pod-reader" is forbidden: attempt to grant extra privileges:
    ```

    There is currently a known issue where IAM Service Accounts are not
    automatically granted cluster admin authorization. To correct the issue:

    ```sh
    a-kubectl create clusterrolebinding cluster-admin-binding \
        --clusterrole=cluster-admin \
        --user=$primary_account
    ```

3. Create a role binding to associate the role with GCP Service Account 1.
    ```sh
    a-kubectl create rolebinding account1-pod-reader-binding \
        --role=pod-reader \
        --user=$account1 \
        --namespace=ns-1
    ```
4. Try again.
    ```sh
    1-kubectl get pods --namespace=ns-1
    ```

## Service Account 2 Listing Pods

1. Try to list the pods in `ns-2` using GCP Service Account 2.
    ```sh
    2-kubectl get pods --namespace=ns-2
    ```
2. Create the `pod-reader` role in `ns-2`, and then create a role binding to
   associate the role with GCP Service Account 2.

    ```sh
    a-kubectl create role pod-reader \
        --verb=get \
        --verb=list \
        --verb=watch \
        --resource=pods \
        --namespace=ns-2

    a-kubectl create rolebinding account2-pod-reader-binding \
        --role=pod-reader \
        --user=$account2 \
        --namespace=ns-2
    ```
3. Try to list the pods in `ns-2` using GCP Service Account 2.
    ```sh
    2-kubectl get pods --namespace=ns-2
    ```
    View existing roles:
    ```sh
    a-kubectl get roles --all-namespaces
    ```
    Note: There is not a lot of static checking around roles and role bindings.
    In fact, even this is possible:
    ```sh
    a-kubectl create rolebinding account2-fallacious-binding \
        --role=fallacious-role \
        --user=$account2 \
        --namespace=ns-2
    ```
4. Re-use the role that was created in the namespace where it was created:
    ```sh
    2-kubectl get pods --namespace=ns-1
    ```

    ```sh
    a-kubectl create rolebinding account2-pod-reader-binding \
        --role=pod-reader \
        --user=$account2 \
        --namespace=ns-1
    ```
5. Try again.
    ```sh
    2-kubectl get pods --namespace=ns-1
    ```

