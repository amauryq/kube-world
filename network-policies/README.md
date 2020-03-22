# NetworkPolicies

## In order to use NetworkPolicies in the cluster, we need to have a network plugin that supports them. We can accomplish this alongside an existing flannel setup using canal

```bash
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml

kubectl apply -f canal.yaml
```

## Create a sample nginx pod

```bash
kubectl apply -f network-policy-secure-pod.yml
```

## Create a client pod which can be used to test network access to the Nginx pod

```bash
kubectl apply -f network-policy-client-pod.yml
```

## Use this command to get the cluster IP address of the Nginx pod