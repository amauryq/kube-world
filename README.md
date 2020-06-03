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
# misc
# Here's an example of copying a local file to a container. The syntax follows
kubectl cp <filename> <namespace/podname:/path/tofile>
```

## Relevant Documentation

[The Pod of Minerva](https://interactive.linuxacademy.com/diagrams/ThePodofMinerva.html)

[Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects)

[Kubernetes Documentation](https://kubernetes.io/docs/home/)

## Accessing Kubernetes API

```shell
# Using kubectl proxy
kubectl proxy --port=8080
curl http://localhost:8080/api
# directly
# get the api server
kubectl config view
# get the default token
kubectl describe secret
# make the call
curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```
