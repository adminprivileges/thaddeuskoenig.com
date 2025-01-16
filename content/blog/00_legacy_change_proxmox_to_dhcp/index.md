---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Change Proxmox to DHCP Client'
tags:
  - Proxmox
  - dhcp
  - Legacy
---
I don't really like like allowing servers to set their own IP addresses, i think its kinda weird and i like to handle things at the network level so I typically have all my servers as DHCP clients and I set their addresses statically on the network device they're attached to. Unfortunately proxmox doesn't like that so it doesn't include DHCP in the installer, which is fine. Its easily fixed.

## The Fix
Open up /etc/network/interfaces in your text editor of choice. and remove the Gateway and Netmask lines under auto vmbr0. Then replace the line with iface vmbr0 inet dhcp

Your file should read something like
```
...
iface eth0 inet manual

auto vmbr0
iface vmbr0 inet dhcp
  bridge_ports eth0
  bridge_stp off
  bridge_fd
...
```