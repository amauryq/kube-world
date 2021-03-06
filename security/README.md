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
kubectl apply -f busybox.yaml

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

### Documentation - Kubernetes Security Primitives

[Securing the Cluster](https://kubernetes.io/docs/concepts/cluster-administration/cluster-administration-overview/#securing-a-cluster)

[Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

[Administer the Cluster via kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)

## Cluster Authentication and Authorization

Once the API server has determined who you are (whether a pod or a user), the authorization is handled by RBAC. In this lesson, we will talk about roles, cluster roles, role bindings, and cluster role bindings.

```sh
# Create a new namespace
kubectl create ns web

# Create a new role for services
kubectl apply -f role.yaml

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
kubectl apply -f curl-pod.yaml

# Get the pods in the web namespace
kubectl get pods -n web

# Open a shell to the container
kubectl exec -it curlpod -n web -- sh

# Access PersistentVolumes (cluster-level) from the pod
curl localhost:8001/api/v1/persistentvolumes
```

### Documentation - Cluster Authentication and Authorization

[Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
[RoleBinding and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)

## Creating TLS Certificates

A Certificate Authority (CA) is used to generate TLS certificates and authenticate to your API server. We’ll go through certificate requests and generating a new certificate.

```sh
# Find the CA certificate on a pod in your cluster
kubectl exec busybox -- ls /var/run/secrets/kubernetes.io/serviceaccount

# Download the binaries for the cfssl tool
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

# Make the binary files executable
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64

# Move the files into your bin directory
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

# Check to see if you have cfssl installed correctly
cfssl version

# Create a CSR and private key PEM file
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "my-svc.my-namespace.svc.cluster.local",
    "my-pod.my-namespace.pod.cluster.local",
    "172.168.0.24",
    "10.0.34.2"
  ],
  "CN": "my-pod.my-namespace.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF

# Create a CertificateSigningRequest API object
cat <<EOF | kubectl create -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: pod-csr.web
spec:
  groups:
  - system:authenticated
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF

# View the CSRs in the cluster
kubectl get csr

# View additional details about the CSR
kubectl describe csr pod-csr.web

# Approve the CSR
kubectl certificate approve pod-csr.web

# View the certificate within your CSR
kubectl get csr pod-csr.web -o yaml

# Extract and decode your certificate to use in a file
kubectl get csr pod-csr.web -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
```

### Documentation - Creating TLS Certificates

[Managing TLS in the Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

[Bootstrapping TLS for Your kubelets](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)

[How kubeadm Manages Certificates](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)

## Secure Images

Working with secure images is imperative in Kubernetes, as it ensures your applications are running efficiently and protecting you from vulnerabilities. We’ll go through how to set Kubernetes to use a private registry.

```sh
# View where your Docker credentials are stored
sudo vim /home/cloud_user/.docker/config.json

# Log in to the Docker Hub
sudo docker login

# View the images currently on your server
sudo docker images

# Pull a new image to use with a Kubernetes pod
sudo docker pull busybox:1.28.4

# Log in to a private registry using the docker login command
sudo docker login -u podofminerva -p 'otj701c9OucKZOCx5qrRblofcNRf3W+e' podofminerva.azurecr.io

# View your stored credentials
sudo vim /home/cloud_user/.docker/config.json

# Tag an image in order to push it to a private registry
sudo docker tag busybox:1.28.4 podofminerva.azurecr.io/busybox:latest

# Push the image to your private registry
docker push podofminerva.azurecr.io/busybox:latest

# Create a new docker-registry secret
kubectl create secret docker-registry acr --docker-server=https://podofminerva.azurecr.io --docker-username=podofminerva --docker-password='otj701c9OucKZOCx5qrRblofcNRf3W+e' --docker-email=user@example.com

# Modify the default service account to use your new docker-registry secret
kubectl patch sa default -p '{"imagePullSecrets": [{"name": "acr"}]}'

# The YAML for a pod using an image from a private repository
apiVersion: v1
kind: Pod
metadata:
  name: acr-pod
  labels:
    app: busybox
spec:
  containers:
    - name: busybox
      image: podofminerva.azurecr.io/busybox:latest
      command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
      imagePullPolicy: Always

# Create the pod from the private image
kubectl apply -f acr-pod.yaml

# View the running pod
kubectl get pods
```

### Documentation - Secure Images

[Images](https://kubernetes.io/docs/concepts/containers/images/)

[Pull Images from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

[Configure Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

[Add ImagePullSecrets](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account)

[11 Ways (Not) to Get Hacked](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/)

## Defining Security Contexts

Defining security contexts allows you to lock down your containers, so that only certain processes can do certain things. This ensures the stability of your containers and allows you to give control or take it away. We’ll go through how to set the security context at the container level and the pod level.

```sh
# Run an alpine container with default security
kubectl run pod-with-defaults --image alpine --restart Never -- /bin/sleep 999999

# Check the ID on the container
kubectl exec pod-with-defaults id

#Create a pod that runs the container as user
kubectl apply -f alpine-user-context.yaml

# View the IDs of the new pod created with container user permission
kubectl exec alpine-user-context id

# Create a pod that runs the container as non-root
kubectl apply -f alpine-nonroot.yaml

# View more information about the pod error
kubectl describe pod alpine-nonroot

# Create the privileged container pod
kubectl apply -f privileged-pod.yaml

# View the devices on the default container
kubectl exec -it pod-with-defaults ls /dev

# View the devices on the privileged pod container
kubectl exec -it privileged-pod ls /dev

# You can see more devices than non privileged

# Try to change the time on a default container pod
kubectl exec -it pod-with-defaults -- date +%T -s "12:00:00"

# This is not allowed

# Create the pod that will allow you to change the container’s time:
kubectl run -f kernelchange-pod.yaml

# Change the time on a container
kubectl exec -it kernelchange-pod -- date +%T -s "12:00:00"

# View the new date on the container
kubectl exec -it kernelchange-pod -- date

# Create a pod that’s container has capabilities removed
kubectl apply -f remove-capabilities.yaml

# Try to change the ownership of a container with removed capability
kubectl exec remove-capabilities chown guest /tmp

# Create a pod that will not allow you to write to the local container filesystem
kubectl apply -f readonly-pod.yaml

# Try to write to the container filesystem
kubectl exec -it readonly-pod touch /new-file

# Create a file on the volume mounted to the container
kubectl exec -it readonly-pod touch /volume/newfile

# View the file on the volume that’s mounted
kubectl exec -it readonly-pod -- ls -la /volume/newfile

# Create a pod with two containers and different group permissions
kubectl apply -f group-context.yaml

# Open a shell to the first container on that pod
kubectl exec -it group-context -c first sh

# Inside the container, get the id command result
id

# Create a file in the mounted volume and see the permissions
echo file > /volume/file
ls -l /volume
```

### Documentation - Defining Security Contexts

[Security Contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

## Securing Persistent Key Value Store

Secrets are used to secure sensitive data you may access from your pod. The data never gets written to disk because it's stored in an in-memory filesystem (tmpfs). Because secrets can be created independently of pods, there is less risk of the secret being exposed during the pod lifecycle.

```sh
# View the secrets in your cluster
kubectl get secrets

# View the default secret mounted to each pod
kubectl describe pods pod-with-defaults

# View the token, certificate, and namespace within the secret
kubectl describe secret

# Generate a private key for your https server
openssl genrsa -out https.key 2048

# Generate a certificate for the https server
openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj /CN=www.example.com

# Create an empty file to create the secret
touch file

# Create a secret from your private key, cert, and file
kubectl create secret generic example-https --from-file=https.key --from-file=https.cert --from-file=file

# View the YAML from your new secret
kubectl get secrets example-https -o yaml

# Apply the config map and the example-https pod yaml files
kubectl apply -f configmap.yaml
kubectl apply -f example-https.yaml

# Describe the nginx conf via ConfigMap
kubectl describe configmap

# View the cert mounted on the container
kubectl exec example-https -c web-server -- mount | grep certs

# Use port forwarding on the pod to server traffic from 443
kubectl port-forward example-https 8443:443 &
# alternate way of port forwarding is ssh -L <node-port>:localhost:<local-port> <user>@<cluster-ip>

# In anew session, curl the web server to get a response
curl https://localhost:8443 -k
```

### Documentation - Securing Persistent Key Value Store

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

## Additional Info

To prevent unauthorized users from modifying the cluster state, RBAC is used, defining roles and role bindings for a user.
A service account resource is created for a pod to determine how it has control over the cluster state.
For example, the default service account will not allow you to list the services in a namespace

## Create a new namespace named my-ns

```bash
kubectl create ns my-ns
```

## Run the kube-proxy pod in the my-ns namespace

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
