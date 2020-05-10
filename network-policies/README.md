# NetworkPolicies

## In order to use NetworkPolicies in the cluster, we need to have a network plugin that supports them. We can accomplish this alongside an existing flannel setup using canal

```bash
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml

kubectl apply -f canal.yaml
```

## Create a sample nginx pod

```bash
kubectl apply -f network-policy-secure-pod.yaml
```

## Create a client pod which can be used to test network access to the Nginx pod

```bash
kubectl apply -f network-policy-client-pod.yaml
```

## Use this command to get the cluster IP address of the Nginx pod

```bash
kubectl get pod network-policy-secure-pod -o wide
```

## Use the secure pod's IP address to test network access from the client pod to the secure Nginx pod

```bash
kubectl exec network-policy-client-pod -- curl <secure pod cluster ip address>
```

## Get information about NetworkPolicies in the cluster

```bash
kubectl get networkpolicies
kubectl describe networkpolicy my-network-policy
```

## Configuring Network Policies

Network policies allow you to specify which pods can talk to other pods. This helps when securing communication between pods, allowing you to identify ingress and egress rules. You can apply a network policy to a pod by using pod or namespace selectors. You can even choose a CIDR block range to apply the network policy.

```sh
# Download the canal plugin. Network policies require this
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml

# Set apiVersion for DaemonSet to apps/v1
# Apply the canal plugin
kubectl apply -f canal.yaml

# Deny communications to all pods in the namespace
kubectl apply -f deny-all-netpolicy.yaml

# Run a deployment to test the NetworkPolicy
kubectl apply -f nginx.yaml

# Create a service for the deployment
kubectl expose deployment nginx --port=80

# Attempt to access the service by using a busybox interactive pod
kubectl run --generator=run-pod/v1 busybox --rm -it --image=busybox /bin/sh
# This will fail because the deny-all net policy
wget --spider --timeout=1 nginx

# Label a pod to get the NetworkPolicy
kubectl label pods [pod_name] app=db
```

### Documentation

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
[Declare Network Policies](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)
[Default Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-policies)
