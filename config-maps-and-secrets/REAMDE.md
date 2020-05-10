# Passing Configuration Data to your Apps

## Config Maps and Secrets

```sh
# Create a ConfigMap with two keys
kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2

# Get the YAML back out from the ConfigMap
kubectl get configmap appconfig -o yaml

# Create the pod that is passing the ConfigMap data
kubectl apply -f configmap-pod.yaml

# Get the logs from the pod displaying the value
kubectl logs configmap-pod

# Create the ConfigMap volume pod
kubectl apply -f configmap-volume-pod.yaml

# Get the keys from the volume on the container
kubectl exec configmap-volume-pod -- ls /etc/config

# Get the values from the volume on the pod
kubectl exec configmap-volume-pod -- cat /etc/config/key1

# Create the secret
kubectl apply -f appsecret.yaml

# Create the pod that has attached secret data
kubectl apply -f secret-pod.yaml

# Open a shell and echo the environment variable
kubectl exec -it secret-pod -- sh
echo $MY_CERT

# Create the pod with volume attached with secrets
kubectl apply -f secret-volume-pod.yaml

# Get the keys from the volume mounted to the container with the secrets
kubectl exec secret-volume-pod -- ls /etc/certs
```

## Documentation

[Configure Pod ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
