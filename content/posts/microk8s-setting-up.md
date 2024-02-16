---
title: "How to configure a cloud native MicroK8s environment"
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

## Requirements
- A server running Ubuntu 22.04 LTS

## Let's get started

Start by running the following command

``` bash
sudo snap install microk8s --channel=1.29/stable --classic
```

Congratulations! You've installed MicroK8s, let's get started by configuring it to be a more cloud native envionment.

### Setting up permissions
To start with we will need to setup permissions for our user, in a production environment it's best pratice to require running as sudo. For security reasons.

```
sudo usermod -a -G microk8s $USER
sudo mkdir -p ~/.kube
sudo chown -f -R $USER ~/.kube
```

Now reboot the server or type the following command: ```newgrp microk8s```

## Check the status

MicroK8s has a built-in command to display its status. During installation you can use the --wait-ready flag to wait for Kubernetes to be up and running:

``` bash
microk8s status --wait-ready
```

## Setting up services

### Ingress

We need a ingress controller to access our services in a proper way. To enable an nginx-ingress type the following command:

```
microk8s enable ingress
```

### Longhorn

Let's install longhorn using the following values, microk8s is a bit more specific since we are running in a snap package. This should do the job + some extra configurations.

Please configure the replicacounts to match the amount of nodes in your cluster.

If you want to expose it with a LoadBalancer, you need to install metallb.
Please do node, that everyone would be able to access the control panel for Longhorn.

``` yaml
defaultSettings:
  defaultDataPath: "/longhorn"
  replicaSoftAntiAffinity: "false"
  replicaNodeSoftAntiAffinity: "false"

# Persistence settings
persistence:
  defaultClassReplicaCount: 1

# Set the storage class to be the default
storageClass:
  isDefaultClass: true

# UI service type
service:
  ui:
    type: ClusterIP

# Configuring the CSI driver with the correct kubelet root directory for MicroK8s
csi:
  kubeletRootDir: "/var/snap/microk8s/common/var/lib/kubelet"
  attacherReplicaCount: 1 
  provisionerReplicaCount: 1
```

### Metallb
To properly expose our services we need to use MetalLB, since Kubernetes on-premise dosen't come with a LoadBalancer builtin.

A LoadBalancer is mostly designed for cloud environments, due to it's use of multiple ip-addresses.

```
microk8s enable ingress
```

You will be prompted for which IP-scope, you want to use. This can be entered like this. The subnets are , seperated, and the range is defined with a -
```
10.0.0.10-10.0.0.100
```