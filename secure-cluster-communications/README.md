# Configuring Secure Cluster Communications

To prevent unauthorized users from modifying the cluster state, RBAC is used, defining roles and role bindings for a user.
A service account resource is created for a pod to determine how it has control over the cluster state.
For example, the default service account will not allow you to list the services in a namespace

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

## Kubernetes Security Primitives

Expanding on our discussion about securing the Kubernetes cluster, we’ll take a look at service accounts and user authentication. How to create a workstation for you to administer your cluster without logging in to the Kubernetes master server.

```sh
# List the service accounts in your cluster
kubectl get serviceaccounts

# Create a new jenkins service account
kubectl create serviceaccount jenkins

# View the YAML for our service account using the abbreviated version
kubectl get sa jenkins -o yaml

# View the secrets in your cluster
kubectl get secret [secret_name]

# Create a new pod with the service account
kubectl apply -f busybox.yml

# View the cluster config that kubectl uses
kubectl config view

#View the config file
cat ~/.kube/config

# Set new credentials for your cluster
kubectl config set-credentials amauryq --username=amauryq --password=password

# Create a role binding for anonymous users (not recommended)
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous

# SCP the certificate authority to your workstation or server
scp /etc/kubernetes/pki/ca.crt cloud_user@[pub-ip-of-remote-client]:~/

# switch to the remote client
# install kubectl

# Set the cluster address and authentication
kubectl config set-cluster kubernetes --server=https://172.31.38.211:6443 --certificate-authority=ca.crt --embed-certs=true

# Set the credentials for amauryq
kubectl config set-credentials amauryq --username=amauryq --password=password

# Set the context for the cluster
kubectl config set-context kubernetes --cluster=kubernetes --user=amauryq --namespace=default

# Use the context
kubectl config use-context kubernetes

# Run the same commands with kubectl
kubectl get nodes
```

### Documentation

[Securing the Cluster](https://kubernetes.io/docs/concepts/cluster-administration/cluster-administration-overview/#securing-a-cluster)

[Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

[Administer the Cluster via kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
