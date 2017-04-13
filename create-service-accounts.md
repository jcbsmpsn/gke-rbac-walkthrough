# Create Service Accounts

NOTE: You will need to enable the IAM API for some of these commands to work.

In order to demonstrate how permissions work, 3 separate users will be used.
Some special steps will be taken to make all three different users work on the
same logged in account. An alias will be created for each one so that it is easy
to see who is taking the action.

1. Admin - this user has full access to all namespaces in the cluster.
2. Account 1 - this user starts with no access and will be granted increasing
   levels of access to namespace 1.
3. Account 2 - this user starts with no access and will be granted increasing
   levels of access to namespace 2, possibly with some minor access to namespace
   1.

There are different ways to make authenticated entities, or subjects, work with
the cluster. In this case, GCP Service Accounts (different from Kubernetes
Service Accounts) will be used for Account 1 and Account 2.

If at some point in these instructions `gcloud` commands stop responding due to
permissions, you can renew your gcloud authentication:

```sh
gcloud auth list
gcloud config set account $primary_account
```

If you have any problem with `kubectl` commands not connecting to the cluster
correctly, you can refresh `kubectl` credentials for your clusters:

```sh
gcloud container clusters get-credentials permissions-test-cluster
```

## Create Admin Alias

Create the `a-kubectl` alias, an alias to `kubectl` that uses the token of the
GCP project master account to authenticate.

```sh
export primary_account="<primary-email-address-of-gcp-project>"
alias a-kubectl='kubectl --token="$(gcloud auth print-access-token --account=$primary_account)"'
```

## Creating First Service Account

Create the `1-kubectl` alias, an alias to `kubectl` that uses a token associated
with the new `cluster-user-1` GCP Service Account to authenticate.

1. Create a GCP service account.
    ```sh
    gcloud iam service-accounts create cluster-user-1 --display-name=cluster-user-1
    ```
2. Capture the full service account name.
    ```sh
    account1=$(gcloud iam service-accounts list --format='value(email)' --filter='displayName:cluster-user-1')
    ```
3. Create a key for the GCP service account.
    ```sh
    gcloud iam service-accounts keys create --iam-account $account1 cluster-user-1.json
    ```
4. Use the GCP service account key to activate the service account.
    ```sh
    gcloud auth activate-service-account $account1 --key-file=cluster-user-1.json
    ```
5. Create an alias to make it easy to use `kubectl` authenticating as the new
   service account.
    ```sh
    alias 1-kubectl='kubectl --token="$(gcloud auth print-access-token --account=$account1)"'
    ```
6. Reset the active account to be ready for the next steps.
    ```sh
    gcloud config set account $primary_account
    ```

## Creating Second Service Account

Create the `2-kubectl` alias, an alias to `kubectl` that uses a token associated
with the new `cluster-user-2` GCP Service Account to authenticate.

This is just a repetition of the same steps for the second service account.

```sh
gcloud iam service-accounts create cluster-user-2 --display-name=cluster-user-2
account2=$(gcloud iam service-accounts list --format='value(email)' --filter='displayName:cluster-user-2')
gcloud iam service-accounts keys create --iam-account $account2 cluster-user-2.json
gcloud auth activate-service-account $account2 --key-file=cluster-user-2.json
alias 2-kubectl='kubectl --token="$(gcloud auth print-access-token --account=$account2)"'
gcloud config set account $primary_account
```

## Enable GCP IAM Cluster Admin Roles

In order for these new GCP Service Accounts to be able to do anything on
clusters, they must have GCP IAM container engine permissions.

1. Navigate to https://console.cloud.google.com
2. Login
3. Select the GCP project that contains your GKE cluster from the drop down list
   on the top.
4. Expand the left menu and select `IAM & Admin` and the `IAM`.
5. Click `Add`
6. Enter the full email address of the user account that you are using as the
   admin user.
7. Select `Container`, then `Container Engine Admin` from the `Role` menu.
8. Add a second role by selecting `Container`, then `Container Engine Cluster
   Admin` from the `Role` menu.
9. Then click `Add` to add the new IAM roles to your user.

## Disable Configured Authentication

The `~/.kube/config` file contains configuration about clusters your `kubectl`
knows about. The aliases configured above use the `--token` parameter to
authenticate cluster connections using a token associated with the correct
service account. However, if there is a valid `auth-provider` section in the
`~/.kube/config` for your cluster, it will override the `--token` parameter and
all requests will be authenticated using the settings `~/.kube/config`.

So, in this, edit the `~/.kube/config` file and comment out the `auth-provider`
section associated with your test cluster. It should look something like this:

```yaml
- name: gke_username-gke-dev_us-central1-d_permissions-test-cluster
  user:
    #auth-provider:
    #  config:
    #    access-token: ya29.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    #    expiry: 2017-01-01T13:23:15.856385727-07:00
    #  name: gcp
```

## Confirm Outcomes

When the above configuration steps are complete, the admin alias should be able
to list nodes, the other two aliases should not be able to do anything.

```sh
➜  a-kubectl get nodes
NAME                                                  STATUS    AGE       VERSION
gke-permissions-test-clu-default-pool-08b7c523-2lpt   Ready     1h        v1.6.1
gke-permissions-test-clu-default-pool-08b7c523-tj2l   Ready     1h        v1.6.1
gke-permissions-test-clu-default-pool-08b7c523-v2z9   Ready     1h        v1.6.1
```

Observe in the error message that `cluster-user-1` is being refused permission.

```sh
➜  1-kubectl get nodes
Error from server (Forbidden): User "cluster-user-1@username-gke-dev.iam.gserviceaccount.com" cannot list nodes at the cluster scope.: "Required \"container.nodes.list\" permission." (get nodes)
```

Observe in the error message that `cluster-user-2` is being refused permission.

```sh
➜  2-kubectl get nodes
Error from server (Forbidden): User "cluster-user-2@username-gke-dev.iam.gserviceaccount.com" cannot list nodes at the cluster scope.: "Required \"container.nodes.list\" permission." (get nodes)
```

