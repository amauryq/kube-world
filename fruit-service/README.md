## Use the busybox pod to test the legacy service on port 80.

```bash
kubectl exec busybox -- curl $(kubectl get pod fruit-service -o=custom-columns=IP:.status.podIP --no-headers):80
```