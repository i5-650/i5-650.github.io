---
title: "Kubernetes - Base principle"
date: 2024-09-07
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


#### Kube-proxy
This is the last piece of the worker. The kube-proxy is responsible for local cluster networking: it ensures each node gets its own unique IP address and implements local iptables/IPVS rules to route and load-balance traffic.

### Kubernetes DNS
In order to have service discovery, you need an internal DNS. So every K8s cluster have one. 
The k8s DNS need to have a static IP that is reserved for it. So every service can contact the API to register to that DNS.


### Packaging apps for k8s
To run an app into a cluster, you need a few things:
1. wrap your app in a pod.
2. deploy your app via a declarative manifest file.







