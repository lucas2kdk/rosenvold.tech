---
title: "How to firmware update cisco IOS over USB"
date: 2023-08-22
author: "Lucas Christensen"
categories:
  - Networking
  - Cisco
slug: "how-to-firmware-update-cisco-usb"
type: "post"
tags: ['Cisco', 'Networking', 'School']
---

## Required Equipment
- Downloaded firmware files.
- An empty USB drive formatted as FAT32.

## Progress Process**
1. Place your IOS files at the root of the USB drive.
2. If the router is not already turned on, power it on.
3. Connect to the equipment using a console connection.
4. Next, check the router's version by running the following command: 

```console
show version
```
4. Copy the IOS version from the USB drive to the router: 

```console
copy usb0:<IOS file name>.bin flash:
```

5. Then, execute the following commands:

```console
en
conf t
boot system flash:<IOS file name>.bin
```

This tells the equipment where to boot from. 

```console
end
wr
reload
```
6. Once the router has restarted, you should delete the old file: 

```console
delete /recursive /force flash:<Old IOS name>.bin
```

(Note: Ensure to replace placeholder text such as `<IOS file name>` and `<Old IOS name>` with actual names when using the commands.)
