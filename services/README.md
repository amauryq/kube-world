# Services
 
## Create Deployment and expose the deployment's replica pods with a service

```bash
kubectl apply -f nginx-deployment.yml
kubectl apply -f nginx-service.yml
```

## You can get more information about the service with these commands

```bash
kubectl get deployments

kubectl get svc
kubectl get endpoints my-service
```