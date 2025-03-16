---
title: "Why I Love Talos Linux for Kubernetes + Hetzner Robot Setup Guide"
date: 2025-03-16
tags: ["Kubernetes", "Talos", "Hetzner", "Linux", "Cloud", "Infrastructure-as-Code"]
author: "Lucas Christensen"
draft: false
---

So, I love Kubernetes — it's awesome! But let's be honest, maintaining a Kubernetes cluster can sometimes be a bit of a pain. That's where **Talos** really shines. With Talos, I can simplify a lot of that complexity because **everything — including the operating system — is defined as YAML files**.

This approach makes **version-controlling both the OS and Kubernetes configurations easy and consistent**, so rebuilding, scaling, or troubleshooting clusters becomes a much smoother and more predictable process.

### Some key benefits of using Talos:

- **Security by design**: Talos is an immutable and minimal operating system, purpose-built for Kubernetes. There are **no SSH, no shells, and no unnecessary packages**, which **dramatically reduces the attack surface** and makes the OS **highly secure out of the box**.
- **Automated updates and patching**: Keeping both Kubernetes and the underlying OS up to date becomes a **safe and automated process**, reducing downtime and human error.
- **Declarative cluster lifecycle management**: Whether spinning up a new cluster or adding nodes, everything is **repeatable and fully automated** via YAML files.
- **Improved auditability**: Since all configurations are stored as code, it’s easy to **review, audit, and track changes** over time — a huge win for security and compliance.
- **Consistent environments**: Talos ensures that **every node runs the exact same configuration**, removing "snowflake" servers and minimizing configuration drift.

So, in short, **Talos allows me to focus on Kubernetes itself, while giving me better security, stability, and operational simplicity** — all in a fully declarative way.

# Talos Linux on Hetzner Robot — Step-by-Step Guide

This guide will walk you through installing **Talos Linux** on a Hetzner dedicated server using **Hetzner Robot**, including wiping disks, writing the ISO, and bootstrapping Kubernetes.

## 1. Boot into Rescue System

- **Reset** the server into rescue mode from Hetzner Robot.
- **SSH** into the server using the credentials provided by Hetzner.

```bash
ssh root@YOUR_SERVER_IP
```

- Check attached disks:

```bash
lsblk
```

## 2. Disable Software RAID (if enabled)

If software RAID is configured, stop and disable it:

```bash
mdadm --stop /dev/md*
mdadm --zero-superblock /dev/nvme0n1
mdadm --zero-superblock /dev/nvme1n1
```

## 3. Wipe All Disks

To ensure a clean Talos installation, wipe both disks:

```bash
# Delete partitions
sfdisk --delete /dev/nvme0n1
sfdisk --delete /dev/nvme1n1

# Remove filesystem signatures
wipefs -a -f /dev/nvme0n1
wipefs -a -f /dev/nvme1n1
```

> ⚠️ **Double-check disk names** (e.g., `/dev/nvme0n1`, `/dev/sda`) to avoid wiping the wrong disks.

## 4. Download Talos ISO

Download the latest Talos Linux ISO (replace version as needed):

```bash
wget https://github.com/siderolabs/talos/releases/download/v1.9.5/metal-amd64.iso
```

## 5. Write Talos Image to Disk

Write Talos to the selected disk (typically `/dev/nvme0n1`):

```bash
dd if=metal-amd64.iso of=/dev/nvme0n1 bs=4M oflag=direct status=progress
```

> ⚙️ **Note:** Adjust `of=` if you want to install to a different disk.

## 6. Reboot

Once the image is written, **reboot** the server to boot into Talos:

```bash
reboot
```

## 7. Generate Talos Configuration

On your **local machine**, generate the Talos Kubernetes cluster configuration:

```bash
talosctl gen config rosenvold-cluster-k8s https://k8s.rosenvold.tech:6443
```

This will generate:
- `controlplane.yaml`
- `worker.yaml`
- `talosconfig`

## 8. Apply Talos Configuration

From your **local machine**, apply the Talos configuration to the server (replace IP with the actual server IP):

```bash
talosctl apply-config --insecure -n 65.21.25.238 --file controlplane.yaml
```

> ⚙️ **Important:**
Before applying, **edit `controlplane.yaml`** and update the machine disk path:

```yaml
# Example snippet from controlplane.yaml
- device: /dev/nvme0n1  # Make sure this matches your disk
```

## 9. Set Talosctl Endpoints and Nodes

Set the endpoint and node IP for `talosctl` to communicate with your Talos node:

```bash
talosctl config endpoint 65.21.25.238
talosctl config node 65.21.25.238
```

## 10. Launch Talos Dashboard (Optional)

To get a quick visual overview:

```bash
talosctl dashboard
```

## 11. Bootstrap Kubernetes Cluster

From your **local machine**, bootstrap Kubernetes on the control plane node:

```bash
talosctl bootstrap
```

## 12. Retrieve Kubeconfig

Fetch the Kubernetes kubeconfig file:

```bash
talosctl kubeconfig ./kubeconfig
```

Move it to the appropriate directory for `kubectl`:

```bash
mkdir -p ~/.kube
mv ./kubeconfig ~/.kube/config
```

## ✅ Done!

You now have a **running Kubernetes cluster** on Talos Linux at Hetzner!

### Quick Summary of Important Commands:

```bash
# Talos Config Generation
talosctl gen config CLUSTER_NAME https://API_ENDPOINT:6443  

# Apply config to server
talosctl apply-config --insecure -n SERVER_IP --file controlplane.yaml  

# Set endpoints for Talosctl
talosctl config endpoint SERVER_IP
talosctl config node SERVER_IP

# Bootstrap Kubernetes
talosctl bootstrap

# Fetch kubeconfig
talosctl kubeconfig ./kubeconfig
```
