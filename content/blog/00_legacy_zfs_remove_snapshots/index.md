---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'ZFS Remove All Snapshots'
tags:
  - ZFS
  - snapshots
  - Legacy
---
Sometimes it's good to have a nice purge of your ZFS snapshots, whether you're pressed for space or you just feel like doing spring cleaning. The following command will clear all of your ZFS snapshots on your Ubuntu system. 

**Warning: There is no recovery, all the cannonballs have been shot. Only do this if you know you're in a stable state.**

```
sudo zfs list -H -o name -t snapshot | sudo xargs -n1 zfs destroy
```
