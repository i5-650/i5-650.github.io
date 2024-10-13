---
title: "Kubernetes - Let's get into it"
date: 2024-10-12
draft: false
tags:
    - devops
    - kubernetes
    - tooling
    - CI/CD
categories:
    - devops 
---

I have been willing to learn kubernetes for multiples reasons (job, school, personnal projects, etc). And today, I'm writting this blog while I'm learning it!
This blog is mainly for me, to take notes and force myself to understand everything.

## What is Kubernetes or k8s ?
K8s is an application orchestrator. Which means it deploys and manages applications. It is made to respond to changes in order to scale it up / down, self-heal, have zero-downtime, etc.
All you need to do to have this is to set it up correctly in the first place, and it will be autonomous afterward.

> Note: k8s relays on container. Containers allow you to isolate your applications in a package and run it quite everywhere. Think of it as a virutal machine but with only the application and more flexibility.

## Core concepts
Kubernetes wrap it self around core concepts that define its architecture.

### Cluster

A cluster is simply a bunch of machines to host applications. Each machine is a node and they can be physical servers, VM, rpi, etc.
In the cluster, there is a control plane and worker nodes. As the name suggest, the control plane have the "intelligence", it expose an API to assign work to the workers.
The control plane is also responsible for:
- keeping apps healthy
- self-healing
- autoscaling
- etc

> Note: A cluster can have multiple control plane and multiple workers. However, the minimal cluster is two machines: one control plane and one worker.

### Classic roadmap
1. Create a microservice application (make it independant and small).
2. Package your application in a container.
3. Wrap each container in k8s pod. 
4. Deploy your pods to the cluster using controllers (Deployements, DaemonSets, StatefulSets, ConJobs, etc).


## Principle
### The Control Plane

The control plane is a node and it runs a collection of system services. At its simplest form it's a single control plane. This only works if you don't need *High Availability* (HA), which imply that you are in a test environment or on a small lab.
The recommanded nunmber of control plane is 3 to 5 and they should be spread across availability zones.

> Note: It's obviously not a good practice to run user applications on control plane nodes as it's dedicated to managing other nodes.


#### The API Server
It's the back-bone of the cluster. All communication, between all components, must go through the API server. Whether the components are external or internal, each communication must go through the API server.

The API server is a RESTful API that you `POST` YAML configurations files to. Those YAML files are called *Manifests* and they describe the desired state of an application.
Therefore, those manifests contains various informations such as: the container image, the port to expose, the number of pods replicas to run, etc. 

Each request to the server must be authenticated and authorized. After authentication and authorization, the files are validated, persister to the *cluster store* and the changes are scheduled to the worker nodes.

#### The Cluster Store
It's based on [etcd](https://etcd.io/) which is a key-value distributed database. Because it's the only stateful part of the control plane and the single source of truth for the cluster, it's good practice to run between 3 and 5 replicas for HA.
By default, k8s will installs a replica of the the cluster store on every control plane node and moreso configure HA. 

Etcd prefers consistency over availability, which means it halts the updates to maintain consistency. Nevertheless, if this happens, user applications should continue to work but the cluster configuration won't be able to update.

#### The controller manager and controllers
The controller manager is the controller of controllers. It spawns all the core controllers and monitors them to respond to events.
Each of the controller are responsible for a small subset of cluster intelligence and runs as a brackground watch-loop constantly watching the API Server for changes. 

Their goal is to ensure that the *observed state* of the cluster matches the *desired state*. In order to match the desired state, the following logic is implemented for each controller:
1. Obtain desired state 
2. Observe current state
3. Determine differences
4. Reconcile differences

Each controller handle only its own little corner of the cluster. Kind of liek the [SRP](https://www.baeldung.com/java-single-responsibility-principle) in Java.

#### The scheduler
The scheduler watches the API server for new work tasks. When a task arrive, it use a complex logic to filters out nodes incapable of running the task and rank the nodes that are capable to assign the task to the selected one.
If there is not suitable node for the task, the task isn't scheduled and gets marked as pending.

Basically, the scheduler only responsibility is to pick a node to run a task. Nothing more, the running part is someone else job's.

#### Summary
<img src="/kubernetes/control_plane_summary.png" alt="kubernetes control plane schema"/>

### Worker nodes
This is the component where the user application is ran. They are in charge of:
1. watching the API for new work assignments
2. Execute work assignments
3. Report back to the control plane using the API server (because like we said, everyhting is passing through the API)

The workers are much simpler and are in fact made of three major components.

#### Kubelet

The kubelet runs on every works, it's responsible of watching the API for new work task and execute it. 
The second thing the kubelet does it report back to the server. Whether the work task is execute correctly or not, the kubelet just only report to the API. The decision about what to do or what node to move the task to is the control plane responsibility.

#### Container runtime
To perform container related tasks, the kubelet needs a container runtime (pulling images, starting/stopping container, etc).

> Note: In the early days, Kubernetes had native support for Docker. More recently, it’s moved
to a plugin model called the Container Runtime Interface (CRI). At a high-level, the CRI
masks the internal machinery of Kubernetes and exposes a clean documented interface
for 3rd-party container runtimes to plug into.

#### Pods

It is the atomic unit of scheduling. It's the equivalent of a container in Docker. Yes, k8s needs your app to be containerized. And this container will be ran insied a Pod.

> Because Pods are objects in k8s API, you capitalize the first letter. It adds clarity and the official dacs are moving towards this standard.

#### Kube-proxy
This is the last piece of the worker. The kube-proxy is responsible for local cluster networking: it ensures each node gets its own unique IP address and implements local iptables/IPVS rules to route and load-balance traffic.

### Kubernetes DNS
In order to have service discovery, you need an internal DNS. So every K8s cluster have one. 
The k8s DNS need to have a static IP that is reserved for it. So every service can contact the API to register to that DNS.


### Packaging apps for k8s
To run an app into a cluster, you need a few things:
1. wrap your app in a pod.
2. deploy your app via a declarative manifest file.

Which means that you will create a container for your micro-service app, publish that container onto a registry (not necessarly a public one, but one that will be accessible by your nodes) and your pods will pull those images to run it.
You will obviously need to create a pod for that container (we will se more of that later) and you are good to go! 

The most common controller is the *Deployment* because it offers scalability, self-healing, rolling updates, and rollout for stateless apps.
This *Deployment* is defined in a YAML manifest file that describe many things: replicas, how updates are performed, etc.

<img src="/kubernetes/deployment.png"/>

> Note: the pod contains the app and the app dependencies.

### The declarative model and desired state

Those two concept are very important in k8s. And it is like this:
1. Declare the desired state of an app in a manifest file.
2. Post the manifest to the API server
3. K8s stores it in the cluster store as the **desired state**.
4. K8s implements the desired state on the cluster.
5. A controller ensure that the **observed sstate** of the app doesn't vary from the **desired state**.

I won't get into detail of what the manifest looks like right now. It's a bit too abstract (imho) but keep in mind it's just a YAML file that describe something.
However, to post this manifest to the API server, you can simply use the `kubectl` command and everything is quite straight-forward. If you ever get lost in it. Just use `--help`, it's great!
> Note: the `kubectl` send the file over https.

After posting the manifest, k8s will inspect it and identifies which controller to send it to and records the config in the cluster store. It will be part of the overall desired state.
Once this is done, any required work tasks get scheduled to worker nodes where the node component take care of pulling images, starting containers, attaching to networks and starting the app processes.

Ultimately, in background, the controllers run the reconciliation loop that constantly monitor the state of the cluster. If the observed state differ from the desired state, k8s performs the tasks needed to reconcile the issue.


### Pods (again)

We stated the general idea of what a Pod is. Now let's dive a bit deeper into what it is.

#### Pods and containers

The term Pod comes for a [pod of whales](https://www.whalefacts.org/what-is-a-group-of-whales-called/) (a group of whales is called a pod) and, as the Docker logo is whale, K8s ran with the whale concept and that's the reason behind "Pods".

The simplest model is 1 Pod = 1 container. That means we will often interchange "Pod" and "container". However, keep in mind that there are advanced use-cases that will require to run multiple containers in a single Pod.
Powerful examples of multi-container Pods includes:
- Service meshes.
- Web containers supported by a helper container pulling updated content.
- Containers with a tightly coupled log scraper.

To summarize, a k8s Pod is a construct for running one or multiple containers.

#### Pods anatomy

I kind of lied to you (sorry), the Pods don't actually run apps. The Pod is an execution environment to run one or more containers.
It ring-fence an area of the host OS and run one or more containers.

If you run multiple containers into one Pod, they will share some things:
- the network stack (e.g the same IP)
- the volume
- the IPC namespace
- the shared memory
- etc. 

If two containers, in the same Pod, need to communicate, they can use `localhost` interface. Because they are in the same net namespace.

Considering you have tightly coupled containers that might need to share memory / storage, having multiple containers into one Pod is ideal. 
However, if you don't need to tightly couple containers, you should put them in their own Pods and couple them over the network.
The inconvenient is it creates a lot of network traffic, potentially not encrypted as well. In this cases, you should consider using a service mesh to secure traffic between Pods and improve the network observability. (I'm not familiar with service mesh so I can't really tell you more for now)

> Note: Pods are the unit of scaling. As we just stated, you don't put multiple containers into a Pod. So you add pods. As it's the atomic unit in k8s.

#### Pods deployment

As Pods are the atomic unit of k8s, a Pod is ready for service when all it's containers (often only one) are up and running. If there is one container that isn't running, the entire Pod isn't up.
In addition, a single Pod can be scheduled to a single node. Which means, you Pod run entirely on one node. You can replicate this Pod, on another or the same node, but you can't have a container of a Pod running on node A and a second container of the same Pod running on node B. 
This is because a Pod is the atomic unit of k8s once again.

#### Pod lifecycle

Pods are mortals, but if they die unexpectedly, you don't bring them back to life. K8s will create a new one instead. It is the same one, but new, a fresh start with a new ID and new IP. 

> Note: you don't have to worry about the IP, because K8s handle it for you as there is an "internal DNS"

This means that you should not design you microservices as tightly coupled with it's Pod instance. Because the Pod instance is volatile. The idea is that your Pod can die at any time and can be replaced seamlessly by another one in the cluster.

The Pod lifecycle also implies that the Pod is immutable. Once the Pod is running, you can't change it. You never log into your Pods to change them. If there is any change, you should replace all the instances. (In k8s, this is the `apply` comamnd)


### Deployments

You will often deploy Pods using a high-level controller such as *Deployments*, *DaemonSets* and *StatefulSets*. 
Most of the time, you will use *Deployments* which is a wrapper around a Pod adding features like: self-healing, scaling, zero-downtime, rollout and versioned rollbacks.

Each of the high-level controllers use, under the hood, watch loops constantly observing the cluster to ensure the observed state is the desired state.


### Service objects and stable networking

We stated that Pods are mortal, that's a bummer. But is it that much of a problem? In fact, no. K8s will handle those "death" and replace the dead Pods.
This also happens when you are doing some rollout and scalling operations. K8s will provide a zero-downtime but all this actions have an impact: a lot of IP churn.

Thus, Pods are unreliable. Because they can die at any moment. How can we counter balance this? With Services!

Services provide a reliable networking for a set of Pods. How convenient.

<img src="/kubernetes/services-networking.jpg" />

Services provide a "front-end" which is a stable DNS and a "back-end" that will load-balance traffic across a dynamic set of Pods.
The services will observe the come and go of the Pods and automatically updates itself to provide a stable networking endpoint.

Same goes for the added/removed Pods. These changes are seamless. The load-balancing is working on TCP and UDP. 

However, the services don't have any application intelligence. They can't provide host and path routing. If you want to do so, you need an Ingress controller. We will get to it later.

## Hands on kubectl

Finally some practice! So, let's setup everything:
- I'm on mac so I will relay on k3d, docker and [orbstack](https://orbstack.dev/).
- I'm on a laptop and so this article can be followed by anyone who have a computer.
- I will try to put as mamy samples as I can to make everything clear.
- A friend of mine already taught me some k8s stuff as I'm writting this article. So I will also rely on [k9s](https://k9scli.io) but it's not mandatory.

### Create a multi-node k8s cluster using k3d

You will need docker, kubectl and k3d to fake the servers. And now what? Let's check our k3d version:


```bash
[nix-shell:~]$ k3d --version
k3d version v5.7.4
k3s version v1.21.7-k3s1 (default)
```
> Yes, I'm using the nix-shell to have it temporary installed onto me laptop as I don't plan to have it forever.

```bash
[nix-shell:~]$ k3d cluster create mini-cluster --servers 1 --agents 3 --image rancher/k3s:latest
```

So, let's check if it worked:
```bash
[nix-shell:~]$ k3d cluster list
NAME           SERVERS   AGENTS   LOADBALANCER
mini-cluster   1/1       3/3      true
```

Putting appart k3d that is faking our nodes, we can use the kubernetes API:

```bash
[nix-shell:~]$ kubectl get nodes
NAME                        STATUS   ROLES                  AGE     VERSION
k3d-mini-cluster-agent-0    Ready    <none>                 3m14s   v1.31.1+k3s1
k3d-mini-cluster-agent-1    Ready    <none>                 3m14s   v1.31.1+k3s1
k3d-mini-cluster-agent-2    Ready    <none>                 3m14s   v1.31.1+k3s1
k3d-mini-cluster-server-0   Ready    control-plane,master   3m17s   v1.31.1+k3s1
```

> Note: there is A LOT of alternatives, but I wont go through them all... The rest of the blog will, use kubectl so it won't change a thing wether you are on a cloud environment or a fake one.
> At the time I'm writting this, a friend told me that k3d do have some issues with the ports. So, i might switch between k3d and orbstack.

### Kubeconfig

TODO: still a bit abstract for now

When you have kubectl installed on your machine, the config for each cluster is store into the `~/.kube/config` file. 
This config file is not that hard to understand as it's a YAML file. 

### Hands on Pods

Pods are often deployed through a higher-level workload controllers (Deployments, DaemonSets, etc). Controllers infuse Pods with many things (as mentionned before) and have a PodTemplate defining the Pods it deploys and manage.
Yet, even if you almost never interact with Pods directly: it's vital to understand them.


