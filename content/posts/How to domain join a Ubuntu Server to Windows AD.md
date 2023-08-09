---
title: "How to Domain Join a Ubuntu 22.04 LTS Server to Active Directory"
date: 2023-04-25
author: "Lucas Christensen"
categories:
  - Windows Server
  - Ubuntu Server
slug: "domain-join-ubuntu-22-04-lts-to-active-directory"
type: "post"
tags: ['ubuntu', 'server', 'linux']
---

# Connecting a Linux Server to Windows Active Directory
This guide will show you how to join an Ubuntu 22.04 LTS server to an existing Windows Active Directory domain.

## Prerequisites
1. A functional Active Directory on the same network.
2. Active Directory server added as DNS server.

## Install WinBind
```bash
sudo apt -y install winbind libpam-winbind libnss-winbind krb5-config samba-dsdb-modules samba-vfs-modules
```

## Configure Winbind
Edit the `/etc/samba/smb.conf` file:
```bash
sudo vim /etc/samba/smb.conf
```

On line 29, change the `workgroup` to the name of your Active Directory domain and add the following lines:
```bash
[global]
   workgroup = YOUR_DOMAIN_NAME
   realm = YOUR_DOMAIN_NAME.LOCAL
   security = ads
   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   idmap config YOUR_DOMAIN_NAME : backend = rid
   idmap config YOUR_DOMAIN_NAME : range = 10000-999999
   template homedir = /home/%U
   template shell = /bin/bash
   winbind use default domain = true
   winbind offline logon = false
```

Edit the `/etc/nsswitch.conf` file:
```bash
sudo vim /etc/nsswitch.conf

```

On line 7, add `winbind` as shown below:
```bash
passwd:         files systemd winbind
group:          files systemd winbind
```

Edit the `/etc/pam.d/common-session` file:
```bash
sudo vim /etc/pam.d/common-session
```

Add the following line to the end of the file if you need to automatically create a home directory at initial login:
```bash
session optional        pam_mkhomedir.so skel=/etc/skel umask=077
```

Update the DNS setting to refer to your Active Directory server:
Edit the `/etc/netplan/01-netcfg.yaml` file:
```bash
sudo vim /etc/netplan/01-netcfg.yaml
```

Add the following lines:
```bash
nameservers:
  addresses: [AD_SERVER_IP_ADDRESS]
```

Apply the network configuration changes:
```bash
sudo netplan apply
```

## Connect to Active Directory
```bash
sudo net ads join -U Administrator
```

## Test the Connection
To show a list of AD users:	

```bash
wbinfo -u
```

The output should be a list of AD users.
To show domain information:
```bash
net ads info
```

The output should show the AD server details and the domain information.

## Troubleshooting
If you encounter issues, consider the following:
1. Can you ping the AD server?
2. Are you using the AD server as the DNS server?
3. Is the time on the Ubuntu server synchronized with the AD server's time?
4. Verify that the Ubuntu server has a unique hostname on the network.
5. Check the firewall on both the Ubuntu server and the AD server.
6. Check the logs on the Ubuntu server for any errors related to Winbind or Samba.
7. Verify that the DNS records for the AD server are correct and can be resolved from the Ubuntu server.
8. Make sure that the Ubuntu server has a valid IP address configuration and can communicate with the AD server over the network.
9. Check the permissions on the `/etc/krb5.keytab` file.

These troubleshooting steps should help you resolve any issues you encounter when joining an Ubuntu 22.04 LTS server to an Active Directory domain.

## References
- [Ubuntu 20.04 LTS : Samba : Winbind : Server World](https://www.server-world.info/en/note?os=Ubuntu_20.04&p=samba&f=4)
- [Ubuntu netplan documentation](https://netplan.io/examples)
- [Microsoft Active Directory documentation](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Ubuntu documentation on domain joining](https://ubuntu.com/server/docs/service-sssd#domain-join)
- [Samba official documentation](https://www.samba.org/)
- [Red Hat documentation on troubleshooting Winbind](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/troubleshooting-winbind)
- [Ubuntu documentation on troubleshooting Samba](https://help.ubuntu.com/community/Samba/SambaTroubleshooting)

These references were used to create this guide and may be helpful for additional information or troubleshooting steps.