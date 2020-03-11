# CKAD

## Build Your Cluster

### On all servers

Set up the Docker and Kubernetes repositories

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
Install Docker and Kubernetes packages

```bash
sudo apt-get update

sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00

sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

Enable iptables bridge call

```bash
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
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
