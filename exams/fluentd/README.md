# fluentd

## Create a descriptor for the ConfigMap

```bash
kubectl apply -f fluentd-config.yaml
```

## Create the pod

```bash
kubectl apply -f /usr/ckad/adapter-pod.yaml
```

## Verify that the pod starts up:

```bash
kubectl get pod counter
```
