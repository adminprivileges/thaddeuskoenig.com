---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Object type requires hosted I/O in HASOS on ESXI'
tags:
  - HomeAssistant
  - ESXI
  - Virtual Machines
  - Legacy
---
So for some reason that Home Assistant OS virtual machine doesn't really like being on certain versions of Vmware ESXI (typically older versions as I assume they do their testing as close to the newer releases as possible). I believe the issue stems from needing to change your virtual disk to IDE instead of SCSI which causes some underlying hard drive integrity issues that they didn't plan for. Nonetheless, the fix is pretty easy and just takes a couple of minutes on your time and a Linux terminal (or any terminal that can SSH I guess). 

First, you're going to want to ssh to your machine and then navigate to where your virtual disk is held for instance mine is in `/vmfs/volumes/65165c2c-a1999659-adf7-1866d5477f7` From there you're going to want to run the following command to repair your hard disk (replace the value in quotes to your disk image name)
```
vmkfstools -x repair "Home_Assistant_Image.vmdk"
```
So from here, your immediate issue s resolved, but if you want to not run into this again you're going to clone your hard disk and replace it with the new one. You can begin with the following command.
```
vmkfstools -i "Home_Assistant_Image.vmdk" "New_Home_Assistant_Image.vmdk"
```
Now from here enter your VM settings > Add Hard Disk > Existing Hard Disk and choose the new disk image. From there you can delete your old disk and boot your machine up again with no problems and you'll have the added benefit of actually being able to do snapshots now.   