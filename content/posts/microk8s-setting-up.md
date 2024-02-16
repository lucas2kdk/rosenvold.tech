---
title: "Kubernetes the last guide you'll ever need"
date: 2024-02-16
author: "Lucas Christensen"
Description: "How to configure a cloud native MicroK8s environment"
categories:
  - Container
  - Kubernetes
slug: "microk8s-setting-up"
type: "post"
tags: ['Kubernetes']
series: ['Kubernetes']
---

- [Opening words](#opening-words)
- [Kubernetes Cheat Sheet](#kubernetes-cheat-sheet)
  - [Basic Concepts](#basic-concepts)
  - [Common Commands](#common-commands)
    - [kubectl Basics](#kubectl-basics)
    - [Working with Pods](#working-with-pods)
    - [Working with Deployments](#working-with-deployments)
    - [Working with Services](#working-with-services)
    - [Namespaces](#namespaces)
    - [Basic Helm Commands](#basic-helm-commands)
      - [Repository Management](#repository-management)
- [Helm Overview](#helm-overview)
- [Key Concepts](#key-concepts)
- [Kubernetes Base Concepts](#kubernetes-base-concepts)
  - [Services](#services)
  - [Deployments](#deployments)
  - [StatefulSets](#statefulsets)
  - [ReplicaSets](#replicasets)
  - [Ingress](#ingress)
- [Kubernetes Concepts Overview](#kubernetes-concepts-overview)
  - [Services](#services-1)
  - [Deployments](#deployments-1)
  - [StatefulSets](#statefulsets-1)
  - [ReplicaSets](#replicasets-1)
  - [Ingress](#ingress-1)
- [Kubernetes learning resources](#kubernetes-learning-resources)
  - [Videos](#videos)
    - [IBM Kubernetes Videos](#ibm-kubernetes-videos)
  - [Documentation](#documentation)
- [Installation of MicroK8s](#installation-of-microk8s)
  - [Requirements](#requirements)
  - [Installation](#installation)
  - [Making MicroK8s easier to work with](#making-microk8s-easier-to-work-with)
    - [Aliases](#aliases)
    - [Setting up user permissions](#setting-up-user-permissions)
- [Installation of services](#installation-of-services)
  - [Ingress Controller](#ingress-controller)
  - [Storage](#storage)
    - [Longhorn](#longhorn)
    - [Installing Longhorn](#installing-longhorn)
  - [Networking](#networking)
    - [MetalLB](#metallb)
      - [Background Info](#background-info)
      - [What is a LoadBalancer?](#what-is-a-loadbalancer)
      - [Setting Up MetalLB](#setting-up-metallb)

# Opening words

Kubernetes is a constantly changing technology, therefore I will always recommend that if you encounter any errors, you refer to the official documentation.

I will try to actively maintain this guide with the information and knowledge I gather, but the information may become outdated in the future.

At the moment, this guide provides a one-stop look at Kubernetes and how to set up a cloud-native environment.

Just like building with Lego, there are a ton of different ways to set up your cluster and tailor it to your specific needs.

# Kubernetes Cheat Sheet

Kubernetes is an open-source platform for automating deployment, scaling, and operations of application containers across clusters of hosts.

## Basic Concepts

- Pod: Smallest deployable units that can be created, scheduled, and managed.
- Service: An abstract way to expose an application running on a set of Pods as a network service.
- Deployment: Describes the desired state for your application.
- Namespace: Used to partition resources into logically named groups.

## Common Commands

### kubectl Basics

- Get information about various resources:
  - `kubectl get pods`
  - `kubectl get services`
  - `kubectl get deployments`
  - `kubectl get namespaces`

- Describe resource details:
  - `kubectl describe pod <pod-name>`
  - `kubectl describe service <service-name>`

- Create or apply resources:
  - `kubectl apply -f <filename.yaml>`

- Delete resources:
  - `kubectl delete -f <filename.yaml>`
  - `kubectl delete pod <pod-name>`

### Working with Pods

- Create a pod:
  - `kubectl run <pod-name> --image=<image-name>`

- Get logs for a pod:
  - `kubectl logs <pod-name>`

- Executing commands in a pod:
  - `kubectl exec -it <pod-name> -- <command>`

### Working with Deployments

- Create a deployment:
  - `kubectl create deployment <deployment-name> --image=<image-name>`

- Scale a deployment:
  - `kubectl scale deployment <deployment-name> --replicas=<num>`

- Update a deployment:
  - `kubectl set image deployment/<deployment-name> <container-name>=<new-image>`

### Working with Services

- Expose a pod as a new Kubernetes service:
  - `kubectl expose pod <pod-name> --port=<port> --name=<service-name>`

- Expose a deployment as a service:
  - `kubectl expose deployment <deployment-name> --port=<port> --type=<type>`

### Namespaces

- Create a namespace:
  - `kubectl create namespace <namespace-name>`

- List all namespaces:
  - `kubectl get namespaces`

- Run a pod in a specific namespace:
  - `kubectl run <pod-name> --image=<image-name> --namespace=<namespace-name>`

### Basic Helm Commands

- Search for charts in the Helm repository:
  - `helm search repo [keyword]`

- Install a chart:
  - `helm install [release-name] [chart-name]`

- List all releases:
  - `helm list`

- Upgrade a release:
  - `helm upgrade [release-name] [chart-name]`

- Rollback a release to a previous version:
  - `helm rollback [release-name] [revision]`

- Uninstall a release:
  - `helm uninstall [release-name]`

#### Repository Management

- Add a Helm repository:
  - `helm repo add [repo-name] [repo-url]`

- Update information of available charts locally from chart repositories:
  - `helm repo update`

- List chart repositories:
  - `helm repo list`

- Remove a chart repository:
  - `helm repo remove [repo-name]`

# Helm Overview

Helm is the package manager for Kubernetes. It allows developers and operators to easily package, configure, and deploy applications onto Kubernetes clusters. Helm uses a packaging format called charts. A chart is a collection of files that describe a related set of Kubernetes resources. Charts can be stored and shared across teams through chart repositories.

# Key Concepts

- **Chart**: A Helm package that contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster.
- **Repository**: A place where charts can be collected and shared.
- **Release**: An instance of a chart running in a Kubernetes cluster. One chart can be installed multiple times into the same cluster, with each installation resulting in a new release.

# Kubernetes Base Concepts

## Services

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable network access to a set of Pods, providing a single point of access for a set of Pods.

- Types of Services:
  - `ClusterIP`: Exposes the Service on an internal IP in the cluster. Only reachable within the cluster.
  - `NodePort`: Exposes the Service on each Node's IP at a static port. Accessible from outside the cluster.
  - `LoadBalancer`: Exposes the Service externally using a cloud provider's load balancer.
  - `ExternalName`: Maps the Service to the contents of the `externalName` field, returning a CNAME record with its value.

## Deployments

Deployments manage the deployment and scaling of a set of Pods, and provide declarative updates to Pods along with other useful features.

- Use Cases:
  - Create Deployments to roll out ReplicaSets and Pods.
  - Update Deployments to roll out new Docker images or configurations.
  - Rollback to an earlier Deployment revision.
  - Scale up Deployments to handle more load.
  - Pause Deployments to apply fixes.

## StatefulSets

StatefulSets are suited for applications that require stable, unique network identifiers, stable, persistent storage, ordered, graceful deployment and scaling, and ordered, automated rolling updates.

- Key Features:
  - Stable, unique network identifiers.
  - Stable, persistent storage.
  - Ordered, graceful deployment and scaling.
  - Ordered, automated rolling updates.

## ReplicaSets

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. It's recommended to use Deployments instead of directly using ReplicaSets unless you require custom update orchestration.

- Use Cases:
  - Ensure a specified number of pod replicas are running.
  - Use in conjunction with Deployments for updates.

## Ingress

Ingress manages external access to the services in a cluster, typically HTTP. Ingress can provide load balancing, SSL termination, and name-based virtual hosting.

- Key Features:
  - Load balancing.
  - SSL termination.
  - Name-based virtual hosting.

# Kubernetes Concepts Overview

## Services

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable network access to a set of Pods, providing a single point of access for a set of Pods.

- Types of Services:
  - `ClusterIP`: Exposes the Service on an internal IP in the cluster. Only reachable within the cluster.
  - `NodePort`: Exposes the Service on each Node's IP at a static port. Accessible from outside the cluster.
  - `LoadBalancer`: Exposes the Service externally using a cloud provider's load balancer.
  - `ExternalName`: Maps the Service to the contents of the `externalName` field, returning a CNAME record with its value.

## Deployments

Deployments manage the deployment and scaling of a set of Pods, and provide declarative updates to Pods along with other useful features.

- Use Cases:
  - Create Deployments to roll out ReplicaSets and Pods.
  - Update Deployments to roll out new Docker images or configurations.
  - Rollback to an earlier Deployment revision.
  - Scale up Deployments to handle more load.
  - Pause Deployments to apply fixes.

## StatefulSets

StatefulSets are suited for applications that require stable, unique network identifiers, stable, persistent storage, ordered, graceful deployment and scaling, and ordered, automated rolling updates.

- Key Features:
  - Stable, unique network identifiers.
  - Stable, persistent storage.
  - Ordered, graceful deployment and scaling.
  - Ordered, automated rolling updates.

## ReplicaSets

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. It's recommended to use Deployments instead of directly using ReplicaSets unless you require custom update orchestration.

- Use Cases:
  - Ensure a specified number of pod replicas are running.
  - Use in conjunction with Deployments for updates.

## Ingress

Ingress manages external access to the services in a cluster, typically HTTP. Ingress can provide load balancing, SSL termination, and name-based virtual hosting.

- Key Features:
  - Load balancing.
  - SSL termination.
  - Name-based virtual hosting.

# Kubernetes learning resources

## Videos

### IBM Kubernetes Videos

[IBM Has made a fantastic playlist with learning resources, and explaining the different Kubernetes concepts.]("https://www.youtube.com/watch?v=2vMEQ5zs1ko&list=PLOspHqNVtKABAVX4azqPIu6UfsPzSu2YN")

## Documentation
For more detailed information and advanced topics, refer to the [official Kubernetes documentation](https://kubernetes.io/docs/).

# Installation of MicroK8s

My preferred Kubernetes Distribution is currently MicroK8s, it’s used in production by multiple companies. And it’s quickly expanding and being improved upon.

The installation is easy, and fast.

MicroK8s is also a direct fork of the “vanilla” version of Kubernetes, and has a minimal footprint of changes.

## Requirements

- It is recommended to use a server running Ubuntu 22.04 LTS, as it is made by Canonical, the same company maintaining MicroK8s. However, MicroK8s can run on any Linux Distribution that is able to run a snap package.

## Installation

Let’s install MicroK8s on Linux, i will install version 1.29 since it’s the latest stable version when writing.

```bash
sudo snap install microk8s --channel=1.29/stable --classic
```

That’s it! You now have successfully installed MicroK8s and have a Kubernetes cluster up and running.

Let’s wait check the status of our installation, and wait for MicroK8s to be fully up and running.

```bash
sudo microk8s status --wait-ready
```

## Making MicroK8s easier to work with

### Aliases

By default we have to run Microk8s in front of every command, we can come around this by setting up aliases on our server.

```bash
sudo snap alias microk8s.kubectl kubectl
sudo snap alias microk8s.helm helm
```

### Setting up user permissions

Normal users do not have permission to interact with Microk8s directly unless they are part of the microk8s group.

For security reasons, it is recommended to run Microk8s as sudo. This is also required for a proper CIS-compliant setup.

```bash
sudo usermod -a -G microk8s $USER
sudo mkdir -p ~/.kube
sudo chown -f -R $USER ~/.kube
```

# Installation of services

By default, MicroK8s does not come with many services. It only includes the essential components, which is beneficial as it allows us to select and utilize the packages and services we need. Additionally, this minimal setup ensures the cluster remains lightweight, making it suitable for running on IoT devices.

## Ingress Controller

An Ingress Controller in Kubernetes is like a receptionist for your computer applications. Imagine you have a bunch of different services or apps inside your cloud setup (like different departments in a big company). When someone from the outside (like a visitor) wants to talk to one of these services, they go through the receptionist. The receptionist knows exactly where to send them based on what they ask for. This is what an Ingress Controller does—it directs incoming internet traffic to the right service inside your setup.

So a proxy that can handle SSL, HTTP & HTTPS Traffic.

We can enable an nginx ingress controller, within MicroK8s by running the following command.

```bash
sudo microk8s enable ingress
```

## Storage

### Longhorn

Prerequities:

The following services needs to be enabled:

```bash
sudo microk8s enable ingress
```

Before You Install Longhorn:

1. Understand Longhorn: Think of Longhorn as a way to create and manage extra storage spaces for your applications running in Kubernetes. It's like adding more hard drives to your cloud setup and making sure they're used efficiently.
2. Match Replica Counts to Nodes: In Longhorn, "replicas" are copies of your data stored across different places for safety. You'll want to set the number of these copies to match the number of "nodes" (individual computers or servers) in your Kubernetes cluster to ensure your data is spread out evenly and safely.
3. Exposing Longhorn with a LoadBalancer: If you want to access Longhorn from outside the cluster easily, you can use something called a LoadBalancer, which is like a public entrance to your storage. But, be careful—this means anyone can try to access it. To set this up, you'll need to install MetalLB, a tool that helps manage these public entrances.

### Installing Longhorn

Now, let's get to the actual steps to install Longhorn on your MicroK8s setup. Here's a simplified version of what you'll need to type into your terminal (a command-line tool where you type instructions to your computer):

1. Prepare Your Cluster: Make sure your MicroK8s cluster is up and running. You might also need to enable some features that Longhorn requires. You can do this with commands in MicroK8s, but the specifics might vary.
2. Install MetalLB (If Needed): If you decided you want that public entrance (LoadBalancer), you'll first install MetalLB. Please check the guide I’ve written for this below.
3. Install Longhorn: You'll use a set of commands to tell MicroK8s to grab Longhorn and set it up with the configurations you want, like the number of replicas matching your nodes. The command will involve applying a configuration file or using a package manager tailored to Kubernetes, like Helm.

Since we are installing MicroK8s it’s a bit special, so I’ve specified some extra values to allow longhorn to run properly within MicroK8s.

```yaml
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace \
  --set defaultSettings.defaultDataPath="/longhorn" \
  --set csi.kubeletRootDir="/var/snap/microk8s/common/var/lib/kubelet"
```

Thanks to this [article](https://github.com/balchua/do-microk8s/blob/master/docs/longhorn.md) for providing the values for making it work.


## Networking

### MetalLB

#### Background Info

When you're running Kubernetes (a system that helps you manage your applications in the cloud) on your own computers (on-premise), it doesn't automatically come with a way to let the outside world talk to your services. This is where MetalLB comes in.

Think of MetalLB as a friendly doorman who can direct visitors to various departments within a big office. In a cloud environment (like a massive corporate building), there's already a doorman service provided. But, when you're doing this on your own setup (like setting up an office in a home), you need to hire your own doorman, and that's MetalLB.

#### What is a LoadBalancer?

A LoadBalancer acts like a smart system that knows how to direct traffic from the internet to the right service in your Kubernetes setup. It's especially useful when you have multiple services that need to be accessible from the outside. In cloud environments, these LoadBalancers can use lots of different IP addresses to manage this traffic.

#### Setting Up MetalLB

When you decide to set up MetalLB in your Kubernetes cluster running with MicroK8s (a lightweight version of Kubernetes), you're essentially adding this smart doorman to your setup. Here's how you do it in simple steps:

1. Enable MetalLB: You start by telling MicroK8s, "Hey, let's hire that doorman." You do this by typing a command into your terminal (a place where you type instructions to your computer). The command looks like this:
microk8s enable metallb
    
    ```bash
    sudo microk8s enable metallb
    ```
    
2. Choose Your IP Range: After you type the command, MicroK8s will ask, "What IP’s should we use" You'll enter this range in a specific format, telling it the starting and ending IP addresses. For example:
    
    ```bash
    10.0.0.10-10.0.0.100
    ```
    
3. If you want to use multiple subnets we can separate, the subnets with a comma. For example:
    
    ```yaml
    10.0.0.10-10.0.0.100,192.168.1.10,192.168.1.100
    ```