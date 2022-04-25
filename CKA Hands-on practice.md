# CKA Hands-on practice

## Pods

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

## Replicasets

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

## Deployments

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
kubectl create deployment <name> --image=<image-name>
```

Scale its replicas

```sh
kubectl scale deployment --replicas-<x> <deployment-name>
```

Get YAML definition file of a specified deployment

```sh
kubectl create deployment --image=<image-name> <deployment-name> --replicas=<x> --dry-run=client -o yaml > output.yaml
```

## Services

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

## Namespaces

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

## Imperative vs declarative

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

