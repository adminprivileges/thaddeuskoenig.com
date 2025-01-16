---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Proxmox Delete Cluster'
tags:
  - proxmox
  - cluster
  - Legacy
---
Theres no real way to delete a cluster in the gui with out doing a fresh install (that i know of) but the following set of commands should stop the cluster service, force it to local, delete the configs and restart it like new.
```
systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l
rm /etc/pve/corosync.conf
rm -r /etc/corosync/*
killall pmxcfs
systemctl start pve-cluster
```