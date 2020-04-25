# Services
 
## Create Deployment and expose the deployment's replica pods with a service

```bash
kubectl apply -f nginx-deployment.yml
kubectl apply -f nginx-service-nodeport.yml
```

## You can get more information about the service with these commands

```bash
kubectl get deployments

kubectl get svc
kubectl get endpoints nginx-nodeport
```
## Get all the clusters in the default namespace

```bash
kubectl get services --field-selector metadata.namespace=default
```
## Connect from busybox pod to the other pods

```bash
kubectl apply -f busybox.yml
# test conectivity
kubectl exec busybox -- curl <pod-ip-address>:<pod-port>
kubectl exec busybox -- curl <service-ip-address>:<pod-port>
curl <kubernetes-node-ip>:<service-node-port>
```
