---
title: "How to reset Cisco Equipment"
date: 2023-08-22
author: "Lucas Christensen"
categories:
  - Networking
  - Cisco
slug: "how-to-reset-cisco-equipment"
type: "post"
tags: ['Cisco', 'Networking', 'School']
---

## Cisco 4221

### Let's get started
- If the router is not already turned on, power it on.
- Connect to the equipment using a console connection.

**Explanations:**  
`config-register 0x2102` – boot normally (default configuration register setting)  
`config-register 0x2120` – boot into ROM Monitor (ROMMON)  
`config-register 0x2142` – ignore contents of NVRAM (startup-configuration)

- Ensure the router is off and connected to power.
- Connect your machine to the router with a console cable.
- Turn on the router.
- While the router is booting send a break signal.
   - This will vary from console.

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