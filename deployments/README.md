# Deployments

## Working with Deployments

```bash
kubectl get deployments
kubectl get deployment <deployment name>
kubectl describe deployment <deployment name>
kubectl edit deployment <deployment name>
kubectl delete deployment <deployment name>
```

## Perform a rolling update

```bash
kubectl set image deployment/rolling-deployment nginx=nginx:1.7.9 --record
```

## Explore the rollout history of the deployment

```bash
kubectl rollout history deployment/rolling-deployment
kubectl rollout history deployment/rolling-deployment --revision=2
```

## You can roll back to the previous revision like so

```bash
kubectl rollout undo deployment/rolling-deployment
```

## You can also roll back to a specific earlier revision by providing the revision number

```bash
kubectl rollout undo deployment/rolling-deployment --to-revision=1
```
