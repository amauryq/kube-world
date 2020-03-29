# CKAD

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

## Kubernetes API primitives

```bash
kubectl cluster-info
kubectl config view
kubectl config use-context <cluster-name>
kubectl get nodes
kubectl get nodes $node_name
kubectl get nodes $node_name -o yaml
kubectl describe nodes
kubectl describe node $node_name
kubectl get pods -n kube-system
kubectl describe pods --all-namespaces
kubectl get services --all-namespaces
kubectl api-resources -o name
```

## Relevant Documentation

[The Pod of Minerva](https://interactive.linuxacademy.com/diagrams/ThePodofMinerva.html)

[Kubernetes Binaries](https://github.com/kubernetes/kubernetes/releases/tag/v1.18.0)

[Install Minikube](https://github.com/kubernetes/minikube)

[Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects)

[Kubernetes Documentation](https://kubernetes.io/docs/home/)


