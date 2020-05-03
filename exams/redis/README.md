# Redis

## Create PersistentVolume

```bash
kubectl apply -f redis-pv.yml
```

## Create PersistentVolumeClaim

```bash
kubectl apply -f redis-pvc.yml
```

## Create Pod

```bash
kubectl apply -f redis-pod.yml
```

## Verify Pod was Created

```bash
kubectl get pods -o wide
```

## Connect to the container and run the redis-cli

```bash
kubectl exec -it redispod redis-cli
```

## Set the key space server:name and value "redis server"
```bash
SET server:name "redis server"
```

## Run the GET command to verify the value was set

```bash
GET server:name
```

## Exit the redis-cli
```bash
QUIT
```
## Delete the Pod and Repeat from ## Create Pod step
```bash
kubectl delete pod redis-pod
```
