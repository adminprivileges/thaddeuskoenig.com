---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Connecting ProxMox cluster over Tailscale'
tags:
  - ProxMox
  - Tailscale
  - Legacy
---
This tutorial is assuming that you already have Tailscale installed on your Proxmox hosts, if you havent done so look at the [installation for Tailscale on Debaian Bullseye](https://tailscale.com/kb/1038/install-debian-bullseye/). 

## Install
1. Because /etc/hosts has priority in all host lookups we are just going to edit the these files on any of the machines that we would like to connect. To do so you can simply vim /etc/hosts and your new host files will end up looking like this with host being the node you want to start the cluster on and remote being the node that will join the cluster
    ```
    127.0.0.1 localhost.localdomain localhost
    # 192.168.1.12 host.domain.com host (this is the original LAN address)
    111.222.111.222  host.domain.com host
    111.222.111.223 remote.domain.com remote
    ```
2. Make sure that the /etc/host file is present on both machines before you run the pvecm commands.

3. On the Host you will create the cluster with a name you choose
    ```
    # pvecm create <CLUSTER_NAME>
    pve create testcluster
    ```
4. On the remote node you will join the cluster BY HOSTNAME (not IP like the Proxmox Docs show).
    ```
    pvecm add <HOST_HOSTNAME>
    pvecm add host
    ```
5. Done... Thats it.