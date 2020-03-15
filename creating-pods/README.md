# Working with Pods

## Create a pod from the yaml definition file

```bash
kubectl create -f my-pod.yml
```

## Edit a pod by updating the yaml definiton and re-applying it

```bash
kubectl apply -f my-pod.yml
```

## You can also edit a pod like this

```bash
kubectl edit my-pod.yml
```

## You can delete a pod like this

```bash
kubectl delete pod my-pod
```
