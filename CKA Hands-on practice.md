# CKA Hands-on practice

## Introduction

### Pods

Check the pods

```sh
kubectl get pods
```

Run new pod with name and image

```sh
kubectl run <pod-name> --image=<image-name>
```

>```sh
>kubectl run nginx-pod --image=nginx:latest
>```

See details about pod

```sh
kubectl describe pod <pod-name>
```

> Check the pod image
>
> ```sh
> kubectl describe pod nginx-pod | grep -i image
> ```

Learn about the **busybox** https://hub.docker.com/_/busybox, https://stackoverflow.com/questions/33291458/i-thought-i-understood-docker-until-i-saw-the-busybox-docker-image

More details in get pods

```sh
kubectl get pods -o wide
```

Delete pod

```sh
kubectl delete pod <pod-name>
```

Create pod from yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mywebapp1
  labels:
    role: webserver-role
    app: nginx
    tier: frontend
spec:
  containers:
  - name: webserver1
    image: nginx:1.6
```

```sh
kubectl create pod -f <path-to-yaml-file>
```

Get yaml file from from pulled pod, but do not run itwith `run` command

```sh
kubectl run redis --image=redis --dry-run=client -o yaml > <yaml-file>
```

### Replicasets

```sh
kubectl get replicasets
```

```sh
kubectl desrcribe replicaset <replicaset-name>
```

Create replicaset from file

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template: # specifies the template pod to replicate
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

```sh
kubectl apply -f <file.yaml>
```

Delete 

```sh
kubectl delete replicaset <name>
```

Edit the replicaset

```sh
kubectl edit replicaset <name>
```

>  You need to delete all pods, so replica can create new pods with changes.



Change replicas number withlut chaning the definition file.

```sh
kubectl scale replicaset --replicas=<x> <replicaset-name>
```

### Deployments

 ```sh
 kubectl get deployments
 ```

```sh
kubectl describe deployment <name>
```

Create deployment from file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

> Deployment definitions file is the same as replica set, but `kind` is different.

Create deployment 

```sh
kubectl create deployment <name> --image=<image-name> --replicas=<x>
```

Scale its replicas

```sh
kubectl scale deployment --replicas-<x> <deployment-name>
```

Get YAML definition file of a specified deployment

```sh
kubectl create deployment --image=<image-name> <deployment-name> --replicas=<x> --dry-run=client -o yaml > output.yaml
```

### Services

get services

```sh
kubectl get svc
```

more details

```
kubectl descrie service <service-name>
```

Create a new service for deployment

```
kubectl expose deployment <deployment-name> --name=webapp-service --target-port=8080 --type=NodePort --port=8080 --dry-run=client -o yaml > <file-name.yaml>
```

And then:

```sh
kubectl apply -f <file-name.yaml>
```

### Namespaces

list namespaces

```sh
kubectl get ns
```

> count namespaces
>
> ```sh
> kubectl get ns --no-header | wc -l 
> ```

get pod from specific namespace

```sh
kubectl get pods -n <namespace-name> --no-headers
```

run pod in specific namespace

```sh
kubectl run redis --image=redis --namespace==<name>
```

get pods from all namespaces

```sh
kubectl get pods --all-namespaces
```

### Imperative vs declarative

Create pod and a cluster ip service for it

```sh
kubectl run <pod-name> --image=<image-name> --labels=<key>=<value>
kubectl expose --type=ClusterIP --name=<service-name> --port=80 --target-port=80
```

>  Dry run  example
>
> ```sh
> kubectl create deployment redis-deploy --image=redis --namespace=dev-ns --dry-run=client -o yaml > elo.yaml
> ```

## Scheduling

### Manual scheduling

Check running kubernetes components like (kube controller manager, kube-proxy etc.)

```sh
kubectl get pods -n kube-system 
```

Manually schedule pod on the given node

First delete existing pod, then update its definition file with adding `node` key in `spec` with value set as node name.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: nginx
spec:
	containers:
	- image: nginx
	  name: nginx
	node: node01
```

```sh
kubectl delete pod <pod-name>
vi <pod-def-file>
kubectl apply -f <pod-def-file>
```

### Labels and selectors

Show labels available 

```sh
kubectl get pods --show-labels
```

Use selector to filter pods with specific labels

```sh
kubeclt get pods --selector <key>=<value>
```

Same you can do with

```sh
kubectl get pods -l <key>=<value>
```

Filter all object by labels

```sh
kubectl get all -l <key>=<value>
```

Fiter by multiple labels

```sh
kubectl get pods -l <key1>=<value1>,<key2>=<value2>..
```

>```sh
>kubectl et pods --selector env=prod,bu=finance,tier=frontend
>```

> bu - business unit

### Taints and Tolerations

Add a taint for node

```sh
kubectl taint nodes <node-name> <key>=<value>:<taint-effect>
```

**taint-effect** determines what happens to pod if it is intolerate for the taint. Can take 3 values:

- NoSchedule - pod cannot be scheduled on the node
- PreferNoSchedule - kube-scheduler will try to schedule pod on the other node, but its not guaranteed
- NoExecute - pod cannot be scheduled on the node and pods currently placed on node are killed  

>```sh
>kubectl taint nodes node1 app=blue:NoSchedule
>```

To add toleration to the pod just add `tolerations` section to the `spec` in pod definition file.

```yaml
tolerations:
- key: "app"
  operator: "Equal"
  value: "blue"
  effect: "NoSchedule"
```

Get nodes

```sh
kubectl get nodes
```

Check node taints

```sh
kubectl describe node <name> | grep Taints
```

To untaint the node just do:

```sh
kubectl taint nodes <node-name> <key>=<value>:<taint-effect>
```

> Same as taint but `-` at the end

Check on which node the pod is:

```sh
kubectl desrbice pod <name> | grep Node
```

```sh
kubectl get pods -o wide
```

## Node Affinity

check node labels

```sh
kubectl get nodes --show-labels
```

check on which node pods are running

```sh
kubectl get pods -o wide
```

get definitions-file for some deployment

```
kubectl create deployment <name> --image=nginx --replicas=6 --dry-run=client -o yaml > elo.yaml
```

Here is the example to copy into file

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
