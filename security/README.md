# Securing the Kubernetes Cluster

## Kubernetes Security Primitives

Expanding on our discussion about securing the Kubernetes cluster, we’ll take a look at service accounts and user authentication. How to create a workstation for you to administer your cluster without logging in to the Kubernetes master server.

```sh
# List the service accounts in your cluster
kubectl get serviceaccounts

# Create a new jenkins service account using the abbreviated version
kubectl create sa jenkins

# View the YAML for our service account and get the secret name
kubectl get sa jenkins -o yaml

# View the secrets in your cluster
kubectl get secret <secret_name>

# Create a new pod with the service account
kubectl apply -f busybox.yml

# View the cluster config that kubectl uses
kubectl config view

# View the config file where you find similar information than above
cat ~/.kube/config

# Set new credentials for your cluster
kubectl config set-credentials user1 --username=user1 --password=password

# Create a role binding for anonymous users (not recommended in production)
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous

# SCP the certificate authority to your workstation or server
scp /etc/kubernetes/pki/ca.crt user1@<pub-ip-of-remote-client>:~/

# Switch to the remote client

# Install kubectl

# Set the cluster address from the config view command and authentication
kubectl config set-cluster kubernetes --server=<server-url> --certificate-authority=ca.crt --embed-certs=true

# Set the credentials for user1
kubectl config set-credentials user1 --username=user1 --password=password

# Set the context for the cluster
kubectl config set-context kubernetes --cluster=kubernetes --user=user1 --namespace=default

# Use the context
kubectl config use-context kubernetes

# Run the same commands with kubectl
kubectl get nodes
```

### Documentation

[Securing the Cluster](https://kubernetes.io/docs/concepts/cluster-administration/cluster-administration-overview/#securing-a-cluster)

[Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

[Administer the Cluster via kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)

## Cluster Authentication and Authorization

Once the API server has determined who you are (whether a pod or a user), the authorization is handled by RBAC. In this lesson, we will talk about roles, cluster roles, role bindings, and cluster role bindings.

```sh
# Create a new namespace
kubectl create ns web

# Create a new role for services
kubectl apply -f role.yml

# Create a RoleBinding
kubectl create rolebinding test --role=service-reader --serviceaccount=web:default -n web

# Run a proxy for inter-cluster communications from another terminal
kubectl proxy

# Try to access the services in the web namespace
curl localhost:8001/api/v1/namespaces/web/services

# Create a ClusterRole to access PersistentVolumes
kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes

# Create a ClusterRoleBinding for the cluster role
kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=web:default

# Create the pod that will allow you to curl directly from the container
kubectl apply -f curl-pod.yml

# Get the pods in the web namespace
kubectl get pods -n web

# Open a shell to the container
kubectl exec -it curlpod -n web -- sh

# Access PersistentVolumes (cluster-level) from the pod
curl localhost:8001/api/v1/persistentvolumes
```

### Documentation

[Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
[RoleBinding and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)


## Configuring Network Policies

Network policies allow you to specify which pods can talk to other pods. This helps when securing communication between pods, allowing you to identify ingress and egress rules. You can apply a network policy to a pod by using pod or namespace selectors. You can even choose a CIDR block range to apply the network policy.

```sh
# Download the canal plugin. Network policies require this
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml

# Set apiVersion for DaemonSet to apps/v1
# Apply the canal plugin
kubectl apply -f canal.yaml

# Deny communications to all pods i the namespace
kubectl apply -f deny-all-netpolicy.yml

# Run a deployment to test the NetworkPolicy
kubectl apply -f nginx.yml

# Create a service for the deployment
kubectl expose deployment nginx --port=80

# Attempt to access the service by using a busybox interactive pod
kubectl apply -f busybox.yml
kubectl exec -it busybox -- /bin/sh

#wget --spider --timeout=1 nginx
The YAML for a pod selector NetworkPolicy:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 5432
Label a pod to get the NetworkPolicy:

kubectl label pods [pod_name] app=db
```

### Documentation

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
[Declare Network Policies](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)
[Default Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-policies)

## ###############
## Additional Info
## ###############

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

