# Cleanup all Resources

```sh
gcloud iam service-accounts delete cluster-user-1 -q
gcloud iam service-accounts delete cluster-user-2 -q
gcloud container clusters delete permissions-test-cluster
```

