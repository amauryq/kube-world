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
kubectl create -f my-pod.yml
```

## Use the -n flag to specify a namespace when using commands like kubectl get

```bash
kubectl get pods -n my-ns
kubectl get pods --all-namespaces --show-labels
kubectl get pods ngnix -n ngnix-ns -o yaml --export > nginx.yml
kubectl apply -f nginx.yml -n ngnix-ns
```

## You can use various selectors to select different subsets of objects

```bash
kubectl get pods -l app=my-app

kubectl get pods -l environment=production

kubectl get pods -l environment=development

kubectl get pods -l environment!=production

kubectl get pods -l 'environment in (development,production)'

kubectl get pods -l app=my-app,environment=production
```

## You can also use -n to specify a namespace when using kubectl describe

```bash
kubectl describe pod my-ns-pod -n my-ns
```

## Edit a pod by updating the yaml definiton and re-applying it

```bash
kubectl apply -f my-pod.yml
```

## You can also edit a pod like this

```bash
kubectl edit my-pod.yml
```

## Explore how the ConfigMap data interacts with pods and containers

```bash
kubectl logs my-configmap-pod

kubectl logs my-configmap-volume-pod -c <multi_container_pod>

kubectl exec my-configmap-volume-pod -- ls /etc/config

kubectl exec my-configmap-volume-pod -- cat /etc/config/myKey
```

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
