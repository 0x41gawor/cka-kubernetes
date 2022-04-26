# Services

Services enable communication between various components both within and outside of the application. Help connect apps together with other apps or connect with users.



## Types

- NodePort

  - makes internal pod port accessible from the node port

- ClusterIP

  - creates  a virtual IP inside the cluster to enable communication between different services 

    - > Communication between frontend and backend apps

- LoadBalancer

  - load balancer for application in supported cloud providers

    - > E. g. to distribute load across the different web servers in your frontend tier.

### NodePort

![](img/1.png)

**NodePort Service** is a Kubernetes object which listens on the port of the node and forwards requests from that port to pod port. This type of service is known as NodePort.

Maps a port on the node to a port on the pod.

<img src="img/2.png" style="zoom:100%;" />

Each service has its cluster IP and **Port** (2)

**NodePorts** (3) can be from 30000 to 32767

**Target por**t (1) is the port on the pod (visible to the user).

> Yes, lots of iptables in implementation

One service can have matching labels for multiple pods. In this case it balances the load with randomly.



<img src="img/3.png" style="zoom:70%;" />

Of course the service works on multiple nodes.

<img src="img/4.png" style="zoom:70%;" />

### Cluster IP

<img src="img/5.png" style="zoom:50%;" />

You have a set for each tier of the app. All of the pods must communicate with each other. But pod addresses are not static, pods are dying, recreating with new ip etc.

So what is the good way to do it?

<img src="img/6.png" style="zoom:50%;" />

Each service gets an IP (Cluster IP - unique cluster scope IP address).

### LoadBalancer

 NodePort just translates Node port to the Pod port, but we still need to know IP of a node.

<img src="img/7.png" style="zoom:50%;" />

This is not what users want. They want a single url accessible from the Internet.

You can just put new VM with load balancing software and configure it, but on the Cloud Supported platform you can use native Kubernetes load balancer.

## Namespaces

In Kubernetes, *namespaces* provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects *(e.g. Deployments, Services, etc)* and not for cluster-wide objects *(e.g. StorageClass, Nodes, PersistentVolumes, etc)*.

> For example you might want to divide cluster resources between `prod` and `dev`, and to be sure that  while working in `dev` you don't accidentally modify `prod` resources.

Each of namespaces can have its own set of policies deciding who can do what and you can also assing **quota** of resources to a namespace so each namespace is guaranteed a certain amount of resources, but can't use more that its allowed limit.

**DNS**

Inside a namespace objects can to refer to each other by they "*firstname*s"

```sh
mysql.connect("db-service")
```

But to a object outside the namespace you need to use fully qualified name.

```
mysql.connect("db-service.namespace-name.svc.cluster.local")
```

**default namespaces**

Kubernetes within installation creates 3 namepsaces: default, kube-system, kube-public. When you run `kubectl get pods` or any command it happens for `default` namespace. You can specify the target namespace with `--namespace <name>` option.

> You can specify namespace in pod definition file in `metadata` section.

If you don't want to specify namespace in option each time you can switch context

```sh
kubectl config set-context $(kubectl config current context) --namespace=<name>
```

**resource quota**

To specify (limit, guarantee) resources in the namespace create resource quota.

# Imperative vs Declarative

<img src="img/8.png" style="zoom:75%;" />

# Annotations

In metadata section, the one when you add labels you can add non-identifying* data just describing the object. This data is annotations. You can put there app author, build version, help desk email etc.

https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

> *In contrary to labels which are used to identify things.

# Scheduling

## Taints and tolerations

**Analogy**

We have a big farm of potatoes. As you know farmers need to fight with some species of worms. In order to make potatoes not edible by the worms they use some kind of spray (taint in k8s), so the smell will fright off some evil species of worms. We say that these species are **intolerant** to this **taint**. 

On the other hand some species of worms are good for our potatoes and we don't want to fright off them. In order to do that we use taint for which smell these species are **tolerant**.

Take on mind that on our farm we can have multiple kinds of potatoes and we can spray each kind with different **taint**, resulting in different set of worms that are **tolerant** and **intolerant**.

**Kubernetes**

In K8S Taints and Tolerations are used to set restriction on what pods can be scheduled on a node. 

>  So we just spray out nodes with some taint :D

**Example**

Lets say we have 3 worker nodes and 4 pods. By default kube-scheduler balances the pods equally.

But lets say we have a dedicated resources on workerNode1 for our frontend app. 

We can spray our WorkerNode1 with taint. Lets say the `blue`taint.

By default pods are intolerant, so none of the pods can be placed on `blue` sprayed node as no one on them have tolerance for `Taint=blue`.

Now we just need to make our desired pod with frontend app to be tolerant to blue.

**Remember**

This mechanism is only to restrict. It will not guarantee you that pod tolerant to some taint will be always placed on the node tainted with it. 

It does not tell the pod where to go, it just tells node which pods accept.

If your goal is to restrict pods to certain nodes it is achieved through *Affinity* mechanism.

**Master node**

K8s in default setup taints master node, so by default no non-kube-system pod can be placed on master node.

It is a good practice not to load master node with any worker pods 

## Node Selectors

You can label nodes and then use this labels in pod-definition file so kube-sched will read them and use.

>For example you have 1 large PC (node01) and two small PCs and you want your pod to be on the large PC.
>
>```sh
>kubectl label nodes node01 size=large
>```
>
>And then in pod definition in `spec`
>
>```yaml
>nodeSelector:
>  size: large
>```

More complex requirements you can achieve with affinity concept.

## Node Affinity

Node Selector are fine, but what if i want to put a pod on a large or medium node, or i want to put a pod on a not a small node?

Add in `spec` in pod definition file

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoreDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: In
          values:
          - large
          - medium
```

OR

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoreDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: NotIn
          values:
          - small
```

Check docs for more `operator` 

The long sentence is a Affinity type. Check the docs.

`required` - restricts a node for pod

`preffered` - try your best to match expression and if there are no nodes place it where you want

`DuringExecution` vs. `DuringScheduling` tells what happens with running pods.

## Taints and Tolerations & Node Affinity

You can combine these two mechanisms to reach your goal.

- Use Node Affinity in POD to restrict some nodes 
- Use Taints and Tolerations to in NODE restrict some pods 

## Resources

By default kube-scheduler assumes pod requests at least:

- 0.5 cpu
- 256 Mi RAM

But you can change this values in pod-definition file in `spec.containers`

```yaml
resources:
  requests:
    memory: "1Gi"
    cpu: 1
```

By default kubernets limits resources for pod with:

- 1 cpu
- 512 Mi RAM

But you can change the limits in pod-definition in `spec.containers`

```yaml
resources:
  limits:
    memory: "2Gi"
    cpu: 2
```

## Deamon sets

The deamon set ensures that one copy of the pod is always present in all nodes in the cluster.

> Use case is a monitoring agent or a log agent.
>
> K8s uses `kube-proxy` as a deamon set

The deamonset definition file resembles the replicaset definition file.

> Replica set ensures certain amount of pods will be in the cluster, while deamon set ensures one copy of certain pod per node

## Static pods

On your host where you have a container-engine (Docker) you can install only `kubelet` no Kubernetes cluster at all.

And you can specify some path for the `kubelet` when you can put pod-definition files and kubelet will take care of ensuring that one pod per file always exists.

<img src="img/9.png" style="zoom:75%;" />

> You cannot create replicasets, services etc. this way. Remember that `kubelet` works only on pod level and this is the only object it understands.

The pods created this way are called static.

Of course you cannot use `kubectl` to get the running pods. `kubectl` works by `kube-apiserver` which is not installed. Use just `docker ps`.

**but**

But if you will join this host as a node to a k8s cluster the pods will be visible by k8s (visible, but not editable).

**usecase**

since static pods are not dependent on k8s control plane you can use static pod to deploy the control plane components itself :D

Thats how the `kubeadm` sets up a K8s cluster.

**vs deamon sets**

<img src="img/10.png" style="zoom:75%;" />

## Multiple schedulers

K8s is highly extensible and you can have your own scheduler with custom checks in it. You can write own scheduler, package it and deploy in your cluster as default or additional scheduler.

Scheduler have its names and in pod-defitnion you can specify scheduler for pod.

# Logging and monitoring

## Cluster components

There is a Metrics Server in K8s which stores in-memory (not on disk) metrics about nodes, pods etc.

Metrics like consumed RAM, cpus etc.

`kubelet` (which is on each node) has component `cAdvisor` which is responsible for retrieving performance metrics from pods.

You can install Metrics Server with minikube or you can get opensource metrics systems (Prometheus, ElasticSearch etc.) from gihub and deploy them with `kubectl apply -f`

<img src="img/11.png" style="zoom:75%;" />

After you have any metrics system you can use for example `kubectl top node` to show nodes performance or `kubectl top pod`

## Pods

```
kubectl logs -f <pod-name>
```

If pod has multiple containers you will need to specify the container name.

