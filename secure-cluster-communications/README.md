# Configuring Secure Cluster Communications

To prevent unauthorized users from modifying the cluster state, RBAC is used, defining roles and role bindings for a user.
A service account resource is created for a pod to determine how it has control over the cluster state.
For example, the default service account will not allow you to list the services in a namespace

## View the kube-config

```bash
cat .kube/config | more
```

## View the service account token

```bash
kubectl get secrets
```

## Create a new namespace named my-ns

```bash
kubectl create ns my-ns
```

# Run the kube-proxy pod in the my-ns namespace

```bash
kubectl run test --image=chadmcrowell/kubectl-proxy -n my-ns
```

## List the pods in the my-ns namespace

```bash
kubectl get pods -n my-ns
```

## Run a shell in the newly created pod

```bash
kubectl exec -it <name-of-pod> -n my-ns sh
```

## List the services in the namespace via API call

```bash
curl localhost:8001/api/v1/namespaces/my-ns/services
```

## View the token file from within a pod

```bash
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

## List the service account resources in your cluster
```bash
kubectl get serviceaccounts
```
