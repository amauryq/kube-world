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
```

2. Install Docker and Kubernetes packages

```bash
# Update your packages
sudo apt-get update

# Install Docker, kubelet, kubeadm, and kubectl
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00

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
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
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

## High Availability

You can provide high availability for cluster components by running multiple instances — however, some replicated components must remain in standby mode. The scheduler and the controller manager are actively watching the cluster state and take action when it changes. If multiples are running, it creates the possibility of unwarranted duplicates of pods.

## Check environment

1. Which components are in which node

```bash
kubectl get pods -o custom-columns=POD:metadata.name,NODE:spec.nodeName --sort-by spec.nodeName -n kube-system
```

2. View the kube-scheduler YAML and find who is the leader

```bash
kubectl get endpoints kube-scheduler -n kube-system -o yaml
```

## Replicating etcd component

Best to have an odd number and no more than seven for any cluster size

3. Create a stacked etcd topology using kubeadm.

- Download the etcd binaries
- Extract and move binaries to /usr/local/bin
- Create two directories:
   * /etc/etcd
   * /var/lib/etcd
- Create the systemd unit file for etcd
- enable and start etcd service

```bash
kubeadm init --config=kubeadm-config.yaml
```

4. Watch as pods are created

```bash
kubectl get pods -n kube-system -w
```

## Documentation

[Kubernetes Binaries](https://github.com/kubernetes/kubernetes/releases/tag/v1.18.0)

[Creating Highly Available Kubernetes Clusters with kubeadm](https://kubernetes.io/docs/setup/independent/high-availability/)

[Highly Available Topologies in Kubernetes](https://kubernetes.io/docs/setup/independent/ha-topology/)

[Operating a Highly Available etcd Cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)