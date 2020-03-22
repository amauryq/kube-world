# PersistentVolumes and PersistentVolumeClaims

## Create the PersistentVolume

```bash
kubectl apply -f my-pv.yml
```

## Create the PersistentVolumeClaim

```bash
kubectl apply -f my-pvc.yml
```

## We can use kubectl to check the status of existing PVs and PVCs

```bash
kubectl get pv
kubectl get pvc
```

## Create a pod to consume storage resources using a PVC

```bash
kubectl apply -f my-pvc-pod.yml
```
