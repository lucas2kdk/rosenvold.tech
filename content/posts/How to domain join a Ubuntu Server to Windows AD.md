+++
categories = ["Windows Server", "Ubuntu Server"]
date = "2023-04-25"
slug = "Domain join a Ubuntu 22.04 LTS Server to Active Directory"
title = "How to domain join a Ubuntu 22.04 LTS Server to Active Directory"
type = ["posts","post"]
[ author ]
  name = "Lucas Christensen"
+++

---

# Connecting Linux Server to Windows AD

[Ubuntu 20.04 LTS : Samba : Winbind : Server World](https://www.server-world.info/en/note?os=Ubuntu_20.04&p=samba&f=4)

## Forudsætninger:

1. A functional Active Directory on the same network.
2. Active Directory server added as DNS server.

## Install  WinBind

```bash
# apt -y install winbind libpam-winbind libnss-winbind krb5-config samba-dsdb-modules samba-vfs-modules
```

## Configure  Winbind

```bash
# **vim /etc/samba/smb.conf**

# line 29: change NetBIOS Name to AD DS's one and add like follows
[global]
   **workgroup = *ROSENVOLD*

   realm = *ROSENVOLD.LOCAL*
   security = ads
   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   idmap config FD3S01 : backend = rid
   idmap config FD3S01 : range = 10000-999999
   template homedir = /home/%U
   template shell = /bin/bash
   winbind use default domain = true
   winbind offline logon = false**

# **vim /etc/nsswitch.conf**
# line 7: add like follows

passwd:         files systemd **winbind**
group:          files systemd **winbind**

# **vim /etc/pam.d/common-session**
# add to the end if you need (auto create a home directory at initial login)

**session optional        pam_mkhomedir.so skel=/etc/skel umask=077**

# change DNS setting to refer to AD

# **vim /etc/netplan/01-netcfg.yaml**

      **nameservers:
        addresses: [*AD Server IP Addresse*]**

# **netplan apply**
```

\newpage
## Connect to Active Directory

```bash
# **net ads join -U Administrator**
```

## Test the connection

```bash
# show AD user list

# **wbinfo -u**

administrator
guest
krbtgt
serverworld
mssql
ldapusers

# show domain info

# net ads info

LDAP server: 10.0.0.100
LDAP server name: fd3s.srv.world
Realm: SRV.WORLD
Bind Path: dc=SRV,dc=WORLD
LDAP port: 389
Server time: Tue, 19 May 2020 16:04:08 JST
KDC server: 10.0.0.100
Server time offset: 0
Last machine account password change: Tue, 19 May 2020 16:02:46 JST
```

## Troubleshooting

1. Can you ping the server?
2. Are you using the server as DNS?
3. Does the Windows Server have the correct time?