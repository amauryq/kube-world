# Storage

## PersistentVolumes and PersistentVolumeClaims

```bash
# Create the PersistentVolume
kubectl apply -f my-pv.yaml

# Create the PersistentVolumeClaim
kubectl apply -f my-pvc.yaml

# We can use kubectl to check the status of existing PVs and PVCs
kubectl get pv
kubectl get pvc

# Create a pod to consume storage resources using a PVC
kubectl apply -f my-pvc-pod.yaml
```
## MongoDB

```sh
# In the Google Cloud Engine, find the region your cluster is in
gcloud container clusters list

# Using Google Cloud, create a persistent disk in the same region as your cluster
gcloud compute disks create --size=1GiB --zone=us-central1-a mongodb

# Create the pod with disk attached and mounted
kubectl apply -f mongodb-pod.yaml

# See which node the pod landed on
kubectl get pods -o wide

# Connect to the mongodb shell
kubectl exec -it mongodb mongo

# Switch to the mystore database in the mongodb shell
use mystore

# Create a JSON document to insert into the database
db.foo.insert({name:'foo'})

# View the document you just created
db.foo.find()

# Exit from the mongodb shell
exit

# Delete the pod
kubectl delete pod mongodb

# Create a new pod with the same attached disk
kubectl apply -f mongodb-pod.yaml

# Check to see which node the pod landed on
kubectl get pods -o wide

# Drain the node (if the pod is on the same node as before)
kubectl drain [node_name] --ignore-daemonsets

# Once the pod is on a different node, access the mongodb shell again
kubectl exec -it mongodb mongo

# Access the mystore database again
use mystore

# Find the document you created from before
db.foo.find()
```

## Documentation

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

[Configure Persistent Volume Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

[PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

[Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

## Applications with Persistent Storage

```sh
# Create our StorageClass object
kubectl apply -f kubeserve-storageclass-fast.yaml

# View the StorageClass objects in your cluster
kubectl get sc

# Create our PVC
kubectl apply -f kubeserve-pvc.yaml

# View the PVC created in our cluster
kubectl get pvc

# View our automatically provisioned PV
kubectl get pv

# Create our deployment and attach the storage to the pods
kubectl apply -f kubeserve-deployment.yaml

# Check the status of the rollout
kubectl rollout status deployments kubeserve

# Check the pods have been created
kubectl get pods

# Connect to our pod and create a file on the PV
kubectl exec -it [pod-name] -- touch /data/file1.txt

# Connect to our pod and list the contents of the /data directory
kubectl exec -it [pod-name] -- ls /data
```