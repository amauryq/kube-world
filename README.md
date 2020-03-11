# CKAD

## Build Your Cluster

### On all servers

1. Set up the Docker and Kubernetes repositories

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

2. Install Docker and Kubernetes packages

```bash
sudo apt-get update

sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00

sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

3. Enable iptables bridge call

```bash
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
```

### On the Kube master server

4. Initialize the cluster

```bash
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
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
# kube >= 1.16
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/3f7d3e6c24f641e7ff557ebcea1136fdf4b1b6a1/Documentation/kube-flannel.yml
```

### On each Kube node server

7. Join the node to the cluster

```bash
# the full command is given when you init the master
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```

## Verify

9. Verify that all nodes are joined and ready

```bash
kubectl get nodes
```

## Lesson Reference

```bash
kubectl api-resources -o name

kubectl get pods -n kube-system

kubectl get nodes

kubectl get nodes $node_name

kubectl get nodes $node_name -o yaml

kubectl describe node $node_name
```

## Relevant Documentation

[https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects)
