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

## You can read the logs

```bash
kubectl logs my-config-map-pod
```


## You can delete a pod like this

```bash
kubectl delete pod my-pod
```
