# Kubernetes Install

## Build Your Cluster

### On all servers

1. Set up the Docker and Kubernetes repositories

```bash
# Get the Docker gpg key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add the Docker repository
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Get the Kubernetes gpg key
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Add the Kubernetes repository
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

# Update your packages
sudo apt-get update
```

2. Install Docker and Kubernetes packages

```bash
# Install Docker, kubelet, kubeadm, and kubectl
sudo apt-get install -y docker-ce=18.06.3~ce~3-0~ubuntu kubelet=1.18.2-00 kubeadm=1.18.2-00 kubectl=1.18.2-00

# Hold them at the current version
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

3. Enable iptables bridge call

```bash
# Add the iptables rule to sysctl.conf
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

# Enable iptables immediately
sudo sysctl -p
```

### On the Kube master server

4. Initialize the cluster

```bash
# Initialize the cluster (run only on the master)
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16
...
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
```

5. Set up local kubeconfig

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

6. Install Flannel networking

```bash
# Apply Flannel CNI network overlay
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### On each Kube node server

7. Join the node to the cluster

```bash
# the full command is given when you init the master
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```

### Verify

9. Verify that all nodes are joined and ready

```bash
kubectl get nodes
```

### Documentation

[Kubernetes Binaries](https://github.com/kubernetes/kubernetes/releases/tag/v1.18.0)

## High Availability

You can provide high availability for cluster components by running multiple instances — however, some replicated components must remain in standby mode. The scheduler and the controller manager are actively watching the cluster state and take action when it changes. If multiples are running, it creates the possibility of unwarranted duplicates of pods.

### Check environment

1. View the pods in the namespace with a custom view. Which components are in which node?

```bash
kubectl get pods -o custom-columns=POD:metadata.name,NODE:spec.nodeName --sort-by spec.nodeName -n kube-system
```

2. View the kube-scheduler YAML. Find who is the leader?

```bash
kubectl get endpoints kube-scheduler -n kube-system -o yaml
```

3. Create the file kubeadm-config.yml

```bash
kubeadm init --config=kubeadm-config.yaml
```

4. Watch as pods are created

```bash
kubectl get pods -n kube-system -w
```

### Replicating etcd component

Best to have an odd number and no more than seven for any cluster size

Create a stacked etcd topology using kubeadm.

- Download the etcd binaries
- Extract and move binaries to /usr/local/bin
- Create two directories:
   * /etc/etcd
   * /var/lib/etcd
- Create the systemd unit file for etcd
- enable and start etcd service

Once this is done proceed to replicate the otheres Kubernetes components

### Documentation

[Creating Highly Available Kubernetes Clusters with kubeadm](https://kubernetes.io/docs/setup/independent/high-availability/)

[Highly Available Topologies in Kubernetes](https://kubernetes.io/docs/setup/independent/ha-topology/)

[Operating a Highly Available etcd Cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

## Configure Secure Cluster Communications

To prevent unauthorized users from modifying the cluster state, RBAC is used, defining roles and role bindings for a user. A service account resource is created for a pod to determine how it has control over the cluster state. For example, the default service account will not allow you to list the services in a namespace.

### View the kube-config. Check self-signed certificate

```bash
cat .kube/config | more
```

### View the service account token

```bash
kubectl get secrets
```

### RBAC in Action

```bash
# Create a new namespace
kubectl create ns my-ns
# Run the kube-proxy pod in the namespace. This pod will serv as a proxy to the API Server.
kubectl run test --image=chadmcrowell/kubectl-proxy -n my-ns
# List the pods in the namespace
kubectl get pods -n my-ns
# Run a shell in the newly created pod
kubectl exec -it test -n my-ns sh
# List the services in the namespace via API call
curl http://localhost:8001/api/v1/namespaces/my-ns/services
# View the token file from within a pod
cat /var/run/secrets/kubernetes.io/serviceaccount/token
# exit
# List the service account resources in your cluster
kubectl get serviceaccounts
```

## Testing the Kubernetes Cluster

### Checklist

```bash
# Run a simple nginx deployment
kubectl run nginx --image=nginx
# View the deployments in your cluster
kubectl get deployments
# View the pods in the cluster:
kubectl get pods
# Use port forwarding to access a pod directly
kubectl port-forward $pod_name 8081:80
# Get a response from the nginx pod directly
curl --head http://localhost:8081
#View the logs from a pod
kubectl logs $pod_name
# Run a command directly from the container
kubectl exec -it $pod_name -- nginx -v
# Create a service by exposing port 80 of the nginx deployment
kubectl expose pod nginx --port 80 --type NodePort
# List the services in your cluster
kubectl get services
# Get a response from the service
curl -I localhost:$node_port
# List the nodes status
kubectl get nodes
# View detailed information about the nodes
kubectl describe nodes
# View detailed information about the pods
kubectl describe pods
```

### Documentation

[Kebetest](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-testing/e2e-tests.md)

[Test a Juju Cluster](https://kubernetes.io/docs/setup/)

## Upgrading Kubernetes Cluster

kubeadm allows us to upgrade our cluster components in the proper order, making sure to include important feature upgrades we might want to take advantage of in the latest stable version of Kubernetes. In this lesson, we will go through upgrading our cluster from version 1.15.9 to 1.16.6.

```bash
# Get the version of the API server
kubectl version --short

#View the version of kubelet
kubectl describe nodes

# View the version of controller-manager pod
kubectl get pod [controller_pod_name] -o yaml -n kube-system

# Release the hold on versions of kubeadm and kubelet
sudo apt-mark unhold kubeadm kubelet

# Install version 1.16.6 of kubeadm
sudo apt install -y kubeadm=1.16.6-00

# Hold the version of kubeadm at 1.16.6
sudo apt-mark hold kubeadm

# Verify the version of kubeadm
kubeadm version

# Plan the upgrade of all the controller components
sudo kubeadm upgrade plan

# Upgrade the controller components
sudo kubeadm upgrade apply v1.16.6

# Release the hold on the version of kubectl
sudo apt-mark unhold kubectl

# Upgrade kubectl
sudo apt install -y kubectl=1.16.6-00

# Hold the version of kubectl at 1.16.6
sudo apt-mark hold kubectl

# Next two also on workers nodes

# Upgrade the version of kubelet
sudo apt install -y kubelet=1.16.6-00

# Hold the version of kubelet at 1.16.6
sudo apt-mark hold kubelet
```

### Documentation

[Upgrading Kubernetes](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)

[Changelog for v1.16](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.16.md)

## Upgrading the Operating System

When we need to take a node down for maintenance, Kubernetes makes it easy to evict the pods on that node, take it down, and then continue scheduling pods after the maintenance is complete.
Furthermore, if the node needs to be decommissioned, you can just as easily remove the node and replace it with a new one, joining it to the cluster.

```bash
# See which pods are running on which nodes
kubectl get pods -o wide

# Evict the pods on a node
kubectl drain [node_name] --ignore-daemonsets

# Watch as the node changes status
kubectl get nodes -w

# Schedule pods to the node after maintenance is complete
kubectl uncordon [node_name]

# Remove a node from the cluster
kubectl delete node [node_name]

# Generate a new token
sudo kubeadm token generate

# List the tokens
sudo kubeadm token list

# Print the kubeadm join command to join a node to the cluster
sudo kubeadm token create [token_name] --ttl 2h --print-join-command

Goto Setup instructions and follow steps for worker
```

### Documentation

[Maintenance on a Node](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#maintenance-on-a-node)

## Backing Up and Restoring a Kubernetes Cluster

Backing up your cluster can be useful, especially if you have a single etcd cluster, as all the cluster state is stored there.
The etcdctl utility allows us to easily create a snapshot of our cluster state (etcd) and save this to an external location.

```bash
# Get the etcd binaries
wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz

# Unzip the compressed binaries
tar xvf etcd-v3.3.12-linux-amd64.tar.gz

# Move the files into /usr/local/bin
sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin

# Take a snapshot of the etcd datastore using etcdctl
sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key

# View the help page for etcdctl
ETCDCTL_API=3 etcdctl --help

# Browse to the folder that contains the certificate files
cd /etc/kubernetes/pki/etcd/

# View that the snapshot was successful
ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db

# Zip up the contents of the etcd directory
sudo tar -zcvf etcd.tar.gz /etc/kubernetes/pki/etcd

Copy the etcd directory to another server
scp etcd.tar.gz user@backup-server:~/
```

### Documentation

[Backing up the etcd Store](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

[etcd Disaster Recovery Examples](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md)
