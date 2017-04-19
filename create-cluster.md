# Create a Cluster

NOTE: Commands for disabling legacy authorization are in the process of being
rolled out and may not be available as you read this. Expect updates within a
week.

You need to have a Kubernetes 1.6 cluster with legacy authorization disabled.

As a transitionary step so that new GKE clusters behave as much as possible as
they did before, legacy authorization and RBAC will both be enabled by default
in new 1.6 clusters. According to how Kubernetes [combines authorization
modes](https://kubernetes.io/docs/admin/authorization/), if either authorization
mode authorizes a requested action, then it will be permitted. In order for RBAC
permissions to be fully enforced and not overridden by the legacy authorization
mechanism, the legacy authorization should be disabled.

## Create a New Cluster

```sh
gcloud beta container clusters create permissions-test-cluster \
    --cluster-version=1.6.1 \
    --no-enable-legacy-authorization
```

## Use an Existing Cluster

1. Verify the version of the existing cluster. The client version and the server
   version must be at least 1.6.
    ```sh
    kubectl version
    ```

2. Disable the legacy authorization mechanism. It is safe to re-run this command
   if the legacy authorization mechanism has already been disabled.
    ```sh
    gcloud beta container clusters update permissions-test-cluster \
        --no-enable-legacy-authorization
    ```

