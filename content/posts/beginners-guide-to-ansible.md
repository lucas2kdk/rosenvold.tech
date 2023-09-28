---
title: "A beginners guide to Ansible"
date: 2023-09-28
author: "Lucas Christensen"
categories:
  - Linux
  - Server
slug: "ansible-for-dummies"
type: "post"
tags: ['linux']
series: ['Networking']
---
This guide is a work in progress, and will be written more to in the future.
## Table of Contents
- [Table of Contents](#table-of-contents)
- [Installation of Ansible](#installation-of-ansible)
  - [Windows](#windows)
  - [Linux](#linux)
    - [Ubuntu](#ubuntu)
  - [macOS](#macos)

## Installation of Ansible

### Windows
To get started with Ansible on Windows start by following my guide on installing WSL2 on Windows since, there's no official support to running Ansible from a Windows machine.
[Installation of WSL2](installation-of-wsl2.md)

Afterwards you can follow the Linux steps below.

### Linux
Ansible should be builtin to your favourite and default package manager. We need to install sshpass to be able to connect to the machines using SSH.

#### Ubuntu
``` bash
sudo apt install update
sudo apt install ansible sshpass
sudo apt -y install python3-paramiko
```

### macOS
First we have to install homebrew on our machine, this can be done faily quickly by doing the following.
Brew is a package manager for macOS, much like the onces you can find builtin to Linux.
```bash
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" 
```

To install ansible type the following command.
``` bash
brew install ansible
```

To install paramiko, which is necessary to run Ansible.
``` bash
pip3 install paramiko
```

Brew dosen't find sshpass secure so we have to install and add a repository for that manually.
``` bash
brew tap esolitos/ipa
brew install esolitos/ipa/sshpass
```