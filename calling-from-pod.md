# Calling the Kubernetes API from a Pod

When an application is running inside a pod, it might be useful for it to access
the Kubernetes API. A pod has a service account, specified in the manifest file
or it gets the default service account associated with the namespace. The
permissions associated with the Kubernetes Service Account are the permissions
that will be given to the applications running in the pod.

The following steps will start a pod with an explicit, non-default, Kubernetes
Service Account, set some permissions and explore the limits of those
permissions from inside a container running inside the pod.

## Start a Pod

```sh
a-kubectl create serviceaccount k-sa-1 --namespace=ns-1
a-kubectl run nginx-1 \
    --image=nginx:latest \
    --namespace=ns-1 \
    --overrides='{
        "spec": {
            "template": {
                "spec": {
                    "serviceAccountName": "k-sa-1"
                }
            }
        }
    }'
```

A `run` command creates a deployment, a replicaset, pods and containers.

You should be able to confirm the success of your deployment:

```sh
➜  a-kubectl get deployments --namespace=ns-1
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-1   1         1         1            1           15m
➜  a-kubectl get replicasets --namespace=ns-1
NAME                 DESIRED   CURRENT   READY     AGE
nginx-1-1297132431   1         1         1         15m
➜  a-kubectl get pods --namespace=ns-1
NAME                       READY     STATUS    RESTARTS   AGE
nginx-1-1297132431-14k2v   1/1       Running   0          15m
```

The pod will have an associated service account which will determine what kind
of permissions it has. To verify the associated service account is the one
request in the run command:

```sh
➜  a-kubectl get pods \
    --namespace=ns-1 \
    -o custom-columns=NAME:metadata.name,SERVICEACCOUNT:spec.serviceAccountName
NAME                       SERVICEACCOUNT
nginx-1-1297132431-14k2v   k-sa-1
```

In order to test the kind of access the pod has, start a shell inside the
running container.
```sh
a-kubectl exec \
    -it $(a-kubectl get pods \
              --namespace=ns-1 \
              -l run=nginx-1 \
              -o custom-columns=:metadata.name | tail -1) \
    --namespace=ns-1 bash
```

`curl` isn't available in this container by default, but it makes a useful tool
for directly exercising the API, so install it.

```sh
apt-get update && apt-get install -y curl
curl -ik https://kubernetes.default.svc.cluster.local/version
```

Note the fully qualified name for the Kubernetes API server endpoint. From
inside a pod running in the default namespace, only `https://kubernetes/` is
required because the API server is running in the default namespace. When trying
to access the API server from a different namespace, the short DNS names from a
different namespace will not resolve, and so the FQDN is necessary.

Now, to do something more interesting, let's list the pods running in our
namespace:

```sh
curl -ik \
     https://kubernetes.default.svc.cluster.local/api/v1/namespaces/ns-1/pods
```

However, listing pods in a namespace requires permissions, so that curl command
should have resulted in an error message, and the error message should have
indicated the `system:anonymous` user was used. So, why aren't requests from
this pod coming from the `k-sa-1` user, which is the service account assigned to
this pod?

Every container in a cluster is populated with a token that can be used for
authenticating to the API server.

```sh
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

When requests are made including this token in the `Authorization` header,
Kubernetes will identify the request as coming from the service account
associated with the pod.

```sh
curl -ik \
     -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
     https://kubernetes.default.svc.cluster.local/api/v1/namespaces/ns-1/pods
```

That curl command should have resulted in an error message as well, but for a
different reason. Now that the correct user is being used, the problem is that
service account doesn't have any permissions to list pods using the API server.

Outside of the container shell, run:

```sh
a-kubectl create role pod-lister \
    --verb=list \
    --resource=pods \
    --namespace=ns-1
a-kubectl create rolebinding pod-lister-binding \
    --role=pod-lister \
    --serviceaccount=ns-1:k-sa-1 \
    --namespace=ns-1
```

Now, back inside the pod, try again:

```sh
curl -ik \
     -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
     https://kubernetes.default.svc.cluster.local/api/v1/namespaces/ns-1/pods
```

Now, something that requires more permissions, listing the pods running in
the default namespace:

```sh
curl -ik \
     -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
     https://kubernetes.default.svc.cluster.local/api/v1/namespaces/default/pods
```

Which fails because we did not grant any pod listing permissions on the default
namespace.
