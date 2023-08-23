---
title: "Factory reset Cisco Equipment"
date: 2023-08-22
author: "Lucas Christensen"
categories:
  - Networking
  - Cisco
slug: "reset-cisco-equipment"
type: "post"
tags: ['Cisco', 'Networking', 'School']
series: ['Networking']
---

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Routers](#routers)
  - [Cisco 4221](#cisco-4221)
  - [Cisco 2901](#cisco-2901)
- [Switches](#switches)
  - [Catalyst 3560 \& 2960](#catalyst-3560--2960)



This guide was written in the assumption, that you have some preknowledge about Cisco Networking.

**Explanations:**  
`config-register 0x2102` – boot normally (default configuration register setting)  
`config-register 0x2120` – boot into ROM Monitor (ROMMON)  
`config-register 0x2142` – ignore contents of NVRAM (startup-configuration)

## Routers

### Cisco 4221

- Ensure the router is off and connected to power.
- Turn on the router.
- Start your console session
- While the router is booting send a break signal.

When arriving at rommon mode enter the following commands:

``` console
rommon 1> confreg 0x2142
rommon 2> reset
```

The router will restart on its own. Once restarted, type:

``` console
en
conf t
config-register 0x2102
end
wr
reload
```

After the router restarts, it's reset, and you can start configuring it.

### Cisco 2901

This is largely the same as resetting a 4221 router, with the exception that the data is stored on a compact flash card, in the front of the router.

- Ensure the router is off and connected to power.
- Remove the compact flash card
- Turn on the router.
- Start your console session
- While the router is booting send a break signal.

- When arriving at rommon mode insert the compact flash card enter the following commands:

``` console
rommon 1> confreg 0x2142
rommon 2> reset
```

- The router will restart on its own. Once restarted, type:

``` console
en
conf t
config-register 0x2102
end
wr
reload
```

- After the router restarts, it's reset, and you can start configuring it.

## Switches

### Catalyst 3560 & 2960

- While the switch is powered on create a console connection.
- Power off the switch.
- While powering on the switch, hold in the MODE button on the front of the switch.

- When the switch is powered on, write the following command.

``` console
flash_init
```

- When the following text appears ```flash_init```, write the following commands.

``` console
del flash:config.text
del flash:vlan.dat
del flash:private-config.text
boot
```

- This will do the following:
  - Delete the config file
  - Delete vlan data
  - In somecases when configuring ports, the switch will save to private-config.text, so we will delete that too.
  - Boot the switch into it's default mode.

Now you have successfully resetted the Switch.