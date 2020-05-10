# Working with Pods

## You can get a list of the namespaces in the cluster like this

```bash
kubectl get namespaces
```

## You can also create your own namespaces

```bash
kubectl create ns my-ns
```

## Create a pod from the yaml definition file

```bash
kubectl create -f my-pod.yaml
```

## Use the -n flag to specify a namespace when using commands like kubectl get

# pods

```bash
kubectl get pods -n my-ns
kubectl get pods --all-namespaces --show-labels
kubectl get pods --all-namespaces [-o wide]
kubectl describe pods --all-namespaces
kubectl get pods ngnix -n ngnix-ns -o yaml --export > nginx.yaml
kubectl apply -f nginx.yaml -n ngnix-ns
```

## You can use various selectors to select different subsets of objects

```bash
# labels
kubectl get pods -l app=my-app
kubectl get pods -l env=pro
kubectl get pods -l env=dev
kubectl get pods -l env!=pro
kubectl get pods -l 'env in (dev,pro)'
kubectl get pods -l app=my-app,env=pro
kubectl get pods -L env
kubectl label pod <pod-name> env=pro
# annotations
kubectl annotate pod <pod-name> my.company/some-annotation="my-company-name"
# field selectors
kubectl get pods --field-selector status.phase=Running,metadata.namespace=default
kubectl get pods --field-selector status.phase=Running,metadata.namespace!=default
```

## You can also use -n to specify a namespace when using kubectl describe

```bash
kubectl describe pod my-ns-pod -n my-ns
```

## Edit a pod by updating the yaml definiton and re-applying it

```bash
kubectl apply -f my-pod.yaml
```

## You can also edit a pod like this

```bash
kubectl edit my-pod.yaml
```

## Explore how the ConfigMap data interacts with pods and containers

```bash
kubectl logs my-configmap-pod
kubectl logs my-configmap-volume-pod -c <multi_container_pod>
kubectl exec -it my-pod -- /bin/bash
kubectl exec my-configmap-volume-pod -- ls /etc/config
kubectl exec my-configmap-volume-pod -- cat /etc/config/myKey
```

## Pod and Node Networking

```bash
# See which node our pod is on
kubectl get pods -o wide

# Log in to the node
ssh [node_name]

# View the node's virtual network interfaces
ifconfig

# View the containers in the pod
docker ps | grep <my-pod>

# Get the process ID for the container
docker inspect --format '{{ .State.Pid }}' [container_id]

#Use nsenter to run a command in the process's network namespace
nsenter -t [container_pid] -n ip addr
```

### Documentation

[Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

## Installing Metrics Server

### Clone the metrics server repo and install the server using kubectl apply

```bash
cd ~/
git clone https://github.com/linuxacademy/metrics-server
kubectl apply -f ~/metrics-server/deploy/1.8+/
```

### Once you have installed the metrics server, you can use this command to verify that it is responsive

```bash
kubectl get --raw /apis/metrics.k8s.io/
```

## Creating a ServiceAccount looks like this

```bash
kubectl create serviceaccount my-serviceaccount
```

## You can delete a pod like this

```bash
kubectl delete pod my-pod
```
