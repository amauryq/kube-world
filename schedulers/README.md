# Kubernetes Scheduler

## Configuring the Kubernetes Scheduler

The default scheduler in Kubernetes attempts to find the best node for your pod by going through a series of steps

```bash
# Label node 1 as being located in availability zone 1
kubectl label node openstackitera2c.mylabserver.com availability-zone=zone1

#Label node 2 as being located in availability zone 2:
kubectl label node openstackitera3c.mylabserver.com availability-zone=zone2

# Label node 1 as dedicated infrastructure:
kubectl label node openstackitera2c.mylabserver.com share-type=dedicated

# Label node 2 as shared infrastructure:
kubectl label node openstackitera3c.mylabserver.com share-type=shared

# Create the deployment
kubectl create -f pref-deployment.yaml

#View the deployment
kubectl get deployments

# View which pods landed on which nodes
kubectl get pods -o wide
```

### Documentation

[Assigning a Pod to a Node](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)

[Pod and Node Affinity Rules](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)


## Scheduling Pods with Taints and Tolerations in Kubernetes

```bash
# Taint one of the worker nodes to repel work
kubectl taint node <node_name> node-type=prod:NoSchedule

kubectl apply -f deployment-2-dev.yaml
kubectl apply -f deployment-2-prod.yaml

# Verify each pod has been scheduled and verify the toleration. No dev pods in tainted node.
kubectl get pods -o wide
```

## Running Multiple Schedulers for Multiple Pods

In Kubernetes, you can run multiple schedulers simultaneously. You can then use different schedulers to schedule different pods. You may, for example, want to set different rules for the scheduler to run all of your pods on one node.

```bash
kubectl apply -f cluster-role.yaml
kubectl apply -f cluster-role-binding.yaml
kubectl apply -f role.yaml
kubectl apply -f role-binding.yaml

# Edit the existing kube-scheduler cluster role
kubectl edit clusterrole system:kube-scheduler

# Add the following
- apiGroups:
  - ""
  resourceNames:
  - kube-scheduler
  - my-scheduler
  resources:
  - endpoints
  verbs:
  - delete
  - get
  - patch
  - update
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - watch
  - list
  - get

kubectl apply -f my-scheduler.yaml

# Schedule pods to schedules

kubectl apply -f pod1.yaml
kubectl apply -f pod2.yaml
kubectl apply -f pod3.yaml

# View the pods as they are created
kubectl get pods -o wide
```

### Documentation

[Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)

## Scheduling Pods with Resource Limits and Label Selectors

```bash
kubectl apply -f resource-pod1.yaml

# Check pod is running
kubectl get pods -o wide

kubectl apply -f resource-pod2.yaml

# See why the pod with a large request didn’t get scheduled
kubectl describe resource-pod2

# Look at the total requests per node
kubectl describe nodes openstackitera3c.mylabserver.com

# Delete the first pod to make room for the pod with a large request
kubectl delete pods resource-pod1

# Watch as the first pod is terminated and the second pod is started
kubectl get pods -o wide -w

# Create pod with limits
kubectl apply -f limited-pod.yaml

# Use the exec utility to use the top command
kubectl exec -it limited-pod top
```
### Documentation

[Configure Default CPU Requests and Limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)

[Configure Default Memory Requests and Limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

## DaemonSets and Manually Scheduled Pods

DaemonSets do not use a scheduler to deploy pods.

```bash
# Find the DaemonSet pods that exist in your kubeadm cluster
kubectl get daemonsets -n kube-system -o wide

# Delete a DaemonSet pod and see what happens
kubectl delete pod [pod_name] -n kube-system

# Create a DaemonSet
kubectl apply -f ssd-monitor.yaml

# Give the node a label to signify it has SSD
kubectl label node [node_name] disk=ssd

# View the DaemonSet pods that have been deployed:
kubectl get daemonset -o wide

# Change or remove the label on a node and watch the DaemonSet pod terminate
kubectl label node [node_name] disk=hdd --overwrite
kubectl label node [node_name] disk-
```

### Docuemntation

[DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

## Displaying Scheduler Events

There are multiple ways to view the events related to the scheduler.

```bash
# View the name of the scheduler pod
kubectl get pods -n kube-system

# Get the information about your scheduler pod events
kubectl describe pods [scheduler_pod_name] -n kube-system

# View the events in your default namespace
kubectl get events

# Delete all the pods in your default namespace
kubectl delete pods --all

# Watch events as they are appearing in real time
kubectl get events -w

# View the logs from the scheduler pod
kubectl logs [kube_scheduler_pod_name] -n kube-system

# The location of a systemd service scheduler pod
cat /var/log/pods/kube-system_kube-scheduler-[node_name]....
```

### Documentation

[Verify the Desired Scheduler](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/#verifying-that-the-pods-were-scheduled-using-the-desired-schedulers)