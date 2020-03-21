# Forwarding Port Traffic with an Ambassador Container

## Create a ConfigMap containing the configuration for the HAProxy ambassador

```bash
kubectl apply -f fruit-service-ambassador-config.yml
```

## Create a multi-container pod which provides access to the legacy service on port 80

```bash
kubectl apply -f fruit-service.yml
```

## Use the busybox pod to test the legacy service on port 80.

```bash
kubectl exec busybox -- curl $(kubectl get pod fruit-service -o=custom-columns=IP:.status.podIP --no-headers):80
```