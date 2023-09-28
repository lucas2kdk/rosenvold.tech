---
title: How to install WSL2
date: 2023-09-28
author: Lucas Christensen
categories:
  - Linux
  - Windows
slug: installation-of-wsl
type: post
tags:
  - ubuntu
  - windows
  - linux
series:
  - Windows Guides
---
**Windows Subsystem for Linux (WSL)** lets you run a full Linux terminal and environment on your Windows machine in no time. This includes Debian, Kali Linux, and Ubuntu!

### Learning Goals

- Enable and install WSL on Windows 10 and Windows 11.
- Install and run a basic graphical application with WSLg.
- Install and run a more advanced application with WSLg.

### Requirements

- A Windows 10 or Windows 11 machine with all updates installed.

## Installing WSL

### Using Command Line

Open a new Powershell prompt as an administrator and type the following command:
``` powershell
wsl --install
```
This command activates the necessary features and installs the default Ubuntu distribution from the Microsoft Store. You’ll need to restart your machine for the changes to take effect.

### Installing Ubuntu

You can install any Linux distribution available on the Windows Store directly from the command line.

In a Powershell terminal, type:

`wsl --list --online` to see all available distributions.

To install a distribution using its NAME, type: `wsl --install -d Ubuntu-22.04`

To check all your installed distributions and their WSL versions, use: `wsl -l -v`
### Setting Up Ubuntu

You’ve got an Ubuntu Terminal on your Windows machine now. Once the setup is done, open the Ubuntu application from your Windows Start menu and set up a username and password.

### (Optional) Turning On systemd

In September 2022, Microsoft [announced systemd support for WSL](https://ubuntu.com/blog/ubuntu-wsl-enable-systemd). This update brings in a lot of handy features for managing processes and services, including snapd support, letting users access all the tools and apps on [snapcraft.io](https://snapcraft.io/store).

To turn on systemd, you need to make a small change to **/etc/wsl.conf** in your Ubuntu distribution.

Type `sudo nano /etc/wsl.conf` to open the file and add the following lines:

``` conf
[boot] systemd=true
```

Then, restart your distribution by typing `wsl --shutdown` in Powershell and opening it again.