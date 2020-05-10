# Services

## Services Types

### ClusterIP

- A ClusterIP exposes the following
* spec.clusterIp:spec.ports[*].port

### NodePort

- A NodePort exposes the following
* <NodeIP>:spec.ports[*].nodePort
* spec.clusterIp:spec.ports[*].port

### Load Balancer

- A LoadBalancer exposes the following
* spec.loadBalancerIp:spec.ports[*].port
* <NodeIP>:spec.ports[*].nodePort
* spec.clusterIp:spec.ports[*].port

## Create Deployment and expose the deployment's replica pods with a service

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service-nodeport.yaml
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
kubectl apply -f busybox.yaml
# test conectivity
kubectl exec busybox -- curl <pod-ip-address>:<pod-port>
kubectl exec busybox -- curl <service-ip-address>:<pod-port>
curl <kubernetes-node-ip>:<service-node-port>
```

## Ingress and Load Balancers

When handling traffic from outside sources, there are two ways to direct that traffic to your pods
- deploying a load balancer
- and creating an ingress controller and an Ingress resource.

```bash
# View the list of services
kubectl get services

# Create a new deployment
kubectl create deployment kubeserve2 --image=chadmcrowell/kubeserve2

# View the list of deployments
kubectl get deployments

# Scale the deployments to 2 replicas
kubectl scale deployment/kubeserve2 --replicas=2

# View which pods are on which nodes
kubectl get pods -o wide

# Create a load balancer from a deployment
kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer

# View the services in your cluster and watch as an external port is created for a service
kubectl get services -w

# Look at the YAML for a service
kubectl get services kubeserve2 -o yaml

# Curl the external IP of the load balancer
curl http://[external-ip]

# View the annotation associated with a service
kubectl describe services kubeserve2

# Set the annotation to route load balancer traffic local to the node
kubectl annotate service kubeserve2 externalTrafficPolicy=Local

# Edit the ingress rules
kubectl edit ingress

# View the existing ingress rules
kubectl describe ingress

# Curl the hostname of your Ingress resource
curl http://kubeserve2.example.com
```

### Documentation

[Create an External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/)

[Ingress]( https://kubernetes.io/docs/concepts/services-networking/ingress/)

## Cluster DNS

CoreDNS is now the new default DNS plugin for Kubernetes. You can customize DNS to include your own nameservers

```bash
# View the CoreDNS obects in the cluster the service that performs load balancing for the DNS Server
kubectl get pods -n kube-system
kubectl get deployments -n kube-system
kubectl get services -n kube-system

kubectl apply -f busybox.yaml

# View the resolv.conf file that contains the nameserver and search in DNS
kubectl exec -it busybox -- cat /etc/resolv.conf

# Look up the DNS name for the native Kubernetes service
kubectl exec -it busybox -- nslookup kubernetes

# Look up the DNS names of your pods in the default namespace
kubectl exec -it busybox -- nslookup [pod-ip-address].default.pod.cluster.local

# Look up a service in your Kubernetes cluster in the kube-system namaspace
kubectl exec -it busybox -- nslookup kube-dns.kube-system.svc.cluster.local

# Get the logs of your CoreDNS pods
kubectl logs [coredns-pod-name]

kubectl apply -f headless-service.yaml
kubectl apply -f custom-dns-pod.yaml

# Check the DNS configuration for that pod
kubectl exec -it dns-sample -- cat /etc/resolv.conf
```

### Documentation

[DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

[Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

[Customizing DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)

[CoreDNS GitHub](https://github.com/coredns/deployment/tree/master/kubernetes)

[Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md)

[Deploying CoreDNS using kubeadm](https://coredns.io/2018/01/29/deploying-kubernetes-with-coredns-using-kubeadm/)
