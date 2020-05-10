# Pizza

## Create Namespace

```bash
kubectl create namespace pizza
```

## Create Deployment

```bash
kubectl apply -f pizza-deployment.yaml
```

## Create Service

```bash
kubectl apply -f pizza-service.yaml
```

## Test

```bash
curl localhost:30080
```
