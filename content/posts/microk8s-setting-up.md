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

Thanks to this article for providing me with the values for making it work.

[](https://github.com/balchua/do-microk8s/blob/master/docs/longhorn.md)

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
    
    ```
    10.0.0.10-10.0.0.100
    ```
    
3. If you want to use multiple subnets we can separate, the subnets with a comma. For example:
    
    ```yaml
    10.0.0.10-10.0.0.100,192.168.1.10,192.168.1.100
    ```