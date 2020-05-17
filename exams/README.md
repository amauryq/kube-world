# CKA Practice Exams

## Exam 1

You have been given access to a three-node cluster. Within that cluster, you will be responsible for creating a service for end users of your web application. You will ensure the application meets the specifications set by the developers and the proper resources are in place to ensure maximum uptime for the app. You must perform the following tasks in order to complete this hands-on lab:

* All objects should be in the web namespace.
* The deployment name should be webapp.
* The deployment should have 3 replicas.
* The deployment’s pods should have one container using the linuxacademycontent/podofminerva image with the tag latest.
* The service should be named web-service.
* The service should forward traffic to port 80 on the pods.
* The service should be exposed externally by listening on port 30080 on each node.
* The pods should be configured to check the /healthz. endpoint on port 8081, and automatically restart the container if the check fails.
* The pods should be configured to not receive traffic until the endpoint on port 80 responds successfully.

### Solution - Exam 1

## Exam 2

You have been given access to a two-node cluster. Within that cluster, a PersistentVolume has already been created. You must identify the size of the volume in order to make a PersistentVolumeClaim and mount the volume to your pod. Once you have created the PVC and mounted it to your running pod, you must copy the contents of /etc/passwd to the volume. Finally, you will delete the pod and create a new pod with the volume mounted in order to demonstrate the persistence of data. You must perform the following tasks in order to complete this hands-on lab:

* All objects should be in the web namespace.
* The PersistentVolumeClaim name should be data-pvc.
* The PVC request should be 256 MiB.
* The access mode for the PVC should be ReadWriteOnce.
* The storage class name should be local-storage.
* The pod name should be data-pod.
* The pod image should be busybox and the tag should be 1.28.
* The pod should request the PersistentVolumeClaim named data-pvc, and the volume name should be temp-data.
* The pod should mount the volume named temp-data to the /tmp/data directory.
* The name of the second pod should be data-pod2.

### Solution - Exam 2

```sh
# data-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
  namespace: web
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi

# create pvc
kubectl apply -f data-pvc.yaml

# data-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-pod
  namespace: web
spec:
  containers:
  - name: data-pod
    image: busybox:1.28
    command: ["cp", "/etc/passwd", "/tmp/data"]
    volumeMounts:
    - mountPath: /tmp/data
      name: temp-data
  restartPolicy: Never
  volumes:
  - name: temp-data
    persistentVolumeClaim:
      claimName: data-pvc

# create pod
kubectl apply -f data-pod.yaml

# data-pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-pod2
  namespace: web
spec:
  containers:
  - name: data-pod2
    image: busybox:1.28
    command: ["ls", "-la", "/tmp/data/passwd"]
    volumeMounts:
    - mountPath: /tmp/data
      name: temp-data
  restartPolicy: Never
  volumes:
  - name: temp-data
    persistentVolumeClaim:
      claimName: data-pvc

# create pod
kubectl apply -f data-pod2.yaml

# view logs
kubectl logs data-pod2 -n web
```

## Exam 3

You have been given access to a three-node cluster. You will be responsible for creating a deployment and a service to serve as a front end for a web application. In addition to the web application, you must deploy a Redis database and make sure the web application can only access this database using the default port of 6379. You will first create a default-deny network policy, so all pods within your Kubernetes are not able to communicate with each other by default. Then you will create a second network policy that specifies the communication on port 6379 between the web application and the database using their label selectors. You must apply these specifications to your resources in order to complete this hands-on lab:

* Create a deployment named webfront-deploy.
* The deployment should use the image nginx with the tag 1.7.8.
* The deployment should expose container port 80 on each pod and contain 2 replicas.
* Create a service named webfront-service and expose port 80, target port 80.
* The service should be exposed externally by listening on port 30080 on each node.
* Create one pod named db-redis using the image redis and the tag latest.
* Verify that you can communicate to pods by default.
* Create a network policy named default-deny that will deny pod communication by default.
* Verify that you can no longer communicate between pods.
* Apply the label role=frontend to the web application pods and the label role=db to the database pod.
* Create a network policy that will apply an ingress rule for the pods labeled with role=db to allow traffic on port 6379 from the pods labeled role=frontend.
* Verify that you have applied the correct labels and created the correct network policies.

### Solution - Exam 3

```sh
# webfront-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webfront-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webfront-deploy
  template:
    metadata:
      name: webfront-deploy
      labels:
        app: webfront-deploy
    spec:
      containers:
      - name: webfront-deploy
        image: nginx:1.7.8
---
apiVersion: v1
kind: Service
metadata:
  name: webfront-service
spec:
  type: NodePort
  selector:
    app: webfront-deploy
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
---
apiVersion: v1
kind: Pod
metadata:
  name: db-redis
  labels:
    app: db-redis
spec:
  containers:
  - name: db-redis
    image: redis:latest

# run busybox and try connecting to service and pods
kubectl run --generator=run-pod/v1 busybox1 -it --image=busybox --label=role=frontend --restart=Never -- /bin/sh

# default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress

# create deny all policy
kubectl apply -f default-deny.yaml

# verify all conections are denied

# apply labels
kubectl label pod db-redis role=db
kubectl label pod webfront[] role=frontend

# allows trafic between pods labeled role=frontend to role=db using port 6379
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-redis
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - port: 6379
```

## Tips for the Certified Kubernetes Administrator (CKA) Exam

```sh
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

kubectl help

kubectl explain

kubectl config use-context
```
