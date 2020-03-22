# Pizza

## Create Namespace

```bash
kubectl create namespace pizza
```

## Create Deployment

```bash
kubectl apply -f pizza-deployment.yml
```

## Create Service

```bash
kubectl apply -f pizza-service.yml
```

## Test

```bash
curl localhost:30080
```
