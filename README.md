# CKAD

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


