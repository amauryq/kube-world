# CKAD

## Kubernetes API primitives

```bash
# cluster
kubectl cluster-info
kubectl get componentstatus
kubectl config view
kubectl config use-context <cluster-name>
kubectl get services --all-namespaces
kubectl api-resources -o name
# nodes
kubectl get nodes [-o wide]
kubectl get nodes $node_name
kubectl get nodes $node_name -o yaml 
kubectl describe nodes
kubectl describe node $node_name
```

## Relevant Documentation

[The Pod of Minerva](https://interactive.linuxacademy.com/diagrams/ThePodofMinerva.html)

[Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects)

[Kubernetes Documentation](https://kubernetes.io/docs/home/)


