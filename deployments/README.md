# Deployments

## Working with Deployments

```bash
kubectl get deployments
kubectl get deployment <deployment name>
kubectl describe deployment <deployment name>
kubectl edit deployment <deployment name>
# If we want to use a different editor
KUBE_EDITOR="nano" kubectl edit deployment <deployment name>
kubectl delete deployment <deployment name>
```

## Deploying an Application, Rolling Updates, and Rollbacks

We already know Kubernetes will run pods and deployments, but what happens when you need to update or change the version of your application running inside of the Kubernetes cluster? That’s where rolling updates come in, allowing you to update the app image with zero downtime.

```bash
# Create a deployment with a record (for rollbacks)
kubectl create -f kubeserve-deployment.yaml --record

# Check the status of the rollout
kubectl rollout status deployments kubeserve

# View the ReplicaSets in your cluster
kubectl get replicasets

# Scale up your deployment by adding more replicas
kubectl scale deployment kubeserve --replicas=5

# Expose the deployment and provide it a service
kubectl expose deployment kubeserve --port 80 --target-port 80 --type NodePort

# When some fix bug or updates are required

# Set the minReadySeconds attribute to your deployment. This will delay the update
kubectl patch deployment kubeserve -p '{"spec": {"minReadySeconds": 10}}'

# Change image to v2

# Use kubectl apply to update a deployment
kubectl apply -f kubeserve-deployment.yaml

# Change image to v1

# Use kubectl replace to replace an existing deployment
kubectl replace -f kubeserve-deployment.yaml

# This update is best done with following

# Run this curl look while the update happens
while true; do curl http://10.105.31.119; done

# Perform the rolling update
kubectl set image deployments/kubeserve app=linuxacademycontent/kubeserve:v2 --v 6

# Describe a certain ReplicaSet
kubectl describe replicasets kubeserve-[hash]

# Apply the rolling update to version 3 (buggy)
kubectl set image deployment kubeserve app=linuxacademycontent/kubeserve:v3

# Undo the rollout and roll back to the previous version
kubectl rollout undo deployments kubeserve

# Look at the rollout history
kubectl rollout history deployment kubeserve

# Look at the rollout history for specific version
kubectl rollout history deployment/kibeserve --revision=2

# Roll back to a certain revision
kubectl rollout undo deployment kubeserve --to-revision=2

# Pause the rollout in the middle of a rolling update (canary release)
kubectl rollout pause deployment kubeserve

# Resume the rollout after the rolling update looks good
kubectl rollout resume deployment kubeserve

# To avoid buggy applications

# Apply the readiness probe
kubectl apply -f kubeserve-deployment-readiness.yaml

# View the rollout status
kubectl rollout status deployment kubeserve

# Describe deployment
kubectl describe deployment
```

### Documentation - Deploying an Application, Rolling Updates, and Rollbacks

[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

[Creating a Deployment](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/)

[Performing a Rolling Update](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

[Scaling Your Application](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#scaling-your-application)

## Working with ReplicaSets and StatefulSets

```sh
# Create the ReplicaSet
kubectl apply -f kubeserver-replicaset.yaml

# Create the pod with the same label
kubectl apply -f kubeserve-pod.yaml

# Watch the pod get terminated
kubectl get pods -w

#Create the StatefulSet
kubectl apply -f statefulset.yaml

# View all StatefulSets in the cluster
kubectl get statefulsets

# Describe the StatefulSets
kubectl describe statefulsets
```

### Documentation - Working with ReplicaSets and StatefulSets

[ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
