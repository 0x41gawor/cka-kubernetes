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

