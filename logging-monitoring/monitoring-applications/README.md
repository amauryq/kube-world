# Monitoring Application Example

```bash
# Play with this 2 pods to see resource consumption
kubectl apply -f resource-consumer-small.yaml
kubectl apply -f resource-consumer-big.yaml

# View resource usage data in the cluster
kubectl top pods
kubectl top pod resource-consumer-big
kubectl top pods -n kube-system
kubectl top nodes
```
