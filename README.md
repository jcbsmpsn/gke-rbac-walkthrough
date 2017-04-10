# gke-rbac-walkthrough

This is not an official Google product.

A walk through of RBAC in Kubernetes 1.6 on a Google Container Engine (GKE)
cluster.

Prerequisites:
1. `gcloud` utility installed
    - It must be version 152+. Run `gcloud version | grep "Google Cloud SDK"` to
      confirm.
    - NOTE: On first publication of these instructions, the necessary version of
      `gcloud` may not be available yet.
1. `kubectl` installed
    - It must be version 1.6+. Run `kubectl version` to confirm.

## Labs

1. [Create a cluster](create-cluster.md)
1. [Create GCP Service Accounts to use for authenticating](create-service-accounts.md)
1. [Create some namespaces](create-namespaces.md)
1. [Create roles and role bindings](create-roles.md)
1. [Cleaning up](cleaning-up.md)
