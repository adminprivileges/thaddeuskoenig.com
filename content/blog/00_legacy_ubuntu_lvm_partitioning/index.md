---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Ubuntu LVM Improper Partitioning'
tags:
  - Ubuntu
  - lvm
  - Virtual Machines
  - Legacy
---
## The Problem
When installing Ubuntu Server with the LVM option in the default partitioning options, it will often only use a fraction of the actual disk space you have available. Thankfully LVM is flexible enough to extend this partition without causing any huge issues to your operating system. In this writeup, I'm gonna extend a 15G volume to a 200G volume on my Ubuntu 22.04.3 LTS VM.

```
user@ubuntu:~$ lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS

loop0                       7:0    0   55M  1 loop /snap/core18/1880

loop1                       7:1    0 55.7M  1 loop /snap/core18/2812

loop2                       7:2    0 63.9M  1 loop /snap/core20/2105

loop3                       7:3    0 74.1M  1 loop /snap/core22/1033

loop4                       7:4    0 71.3M  1 loop /snap/lxd/16099

loop5                       7:5    0 91.8M  1 loop /snap/lxd/24061

loop6                       7:6    0 40.9M  1 loop /snap/snapd/20290

sda                         8:0    0  200G  0 disk 

├─sda1                      8:1    0    1M  0 part 

├─sda2                      8:2    0    1G  0 part /boot

└─sda3                      8:3    0  199G  0 part 

  └─ubuntu--vg-ubuntu--lv 253:0    0   15G  0 lvm  /

sr0                        11:0    1  4.5G  0 rom  



user@ubuntu:~$ df -h

Filesystem                         Size  Used Avail Use% Mounted on

tmpfs                              794M  1.7M  793M   1% /run

/dev/mapper/ubuntu--vg-ubuntu--lv   15G   14G  497M  97% /

tmpfs                              3.9G     0  3.9G   0% /dev/shm

tmpfs                              5.0M     0  5.0M   0% /run/lock

/dev/sda2                          974M  223M  684M  25% /boot

tmpfs                              794M   16K  794M   1% /run/user/1000
```
## The Fix
First, let's ensure that our physical volume is correctly sized, this command won't really do much if it is, but ts worth running just to be sure. 
```
user@ubuntu:~$ sudo pvresize /dev/sda3

  Physical volume "/dev/sda3" changed

  1 physical volume(s) resized or updated / 0 physical volume(s) not resized

user@ubuntu:~$ sudo pvdisplay

  --- Physical volume ---

  PV Name               /dev/sda3

  VG Name               ubuntu-vg

  PV Size               <199.00 GiB / not usable 16.50 KiB

  Allocatable           yes 

  PE Size               4.00 MiB

  Total PE              50943

  Free PE               47104

  Allocated PE          3839

  PV UUID               cJAk5B-AR3A-PYh2-muKW-ZaOm-3T0u-Fo7VWz
```
Now, we need to resize the volume itself to consume 100% of the free space and not just the 15G that it's using now.
```
user@ubuntu:~$ sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv

  Size of logical volume ubuntu-vg/ubuntu-lv changed from <15.00 GiB (3839 extents) to <199.00 GiB (50943 extents).

  Logical volume ubuntu-vg/ubuntu-lv successfully resized.

user@ubuntu:~$ sudo lvdisplay

  --- Logical volume ---

  LV Path                /dev/ubuntu-vg/ubuntu-lv

  LV Name                ubuntu-lv

  VG Name                ubuntu-vg

  LV UUID                n55bW0-FSzX-2CGy-FB4J-8n6G-DpmA-yr2Ll6

  LV Write Access        read/write

  LV Creation host, time ubuntu-server, 2021-07-13 22:47:18 +0000

  LV Status              available

  # open                 1

  LV Size                <199.00 GiB

  Current LE             50943

  Segments               1

  Allocation             inherit

  Read ahead sectors     auto

  - currently set to     256

  Block device           253:0
```
Now we need to go into the file system and extend it with this newfound space. 
```
user@ubuntu:~$ sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

resize2fs 1.46.5 (30-Dec-2021)

Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required

old_desc_blocks = 2, new_desc_blocks = 25



The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 52165632 (4k) blocks long.
```
Now we can check our work. 
```
user@ubuntu:~$ df -h

Filesystem                         Size  Used Avail Use% Mounted on

tmpfs                              794M  1.7M  793M   1% /run

/dev/mapper/ubuntu--vg-ubuntu--lv  196G   14G  175G   8% /

tmpfs                              3.9G     0  3.9G   0% /dev/shm

tmpfs                              5.0M     0  5.0M   0% /run/lock

/dev/sda2                          974M  223M  684M  25% /boot

tmpfs                              794M   16K  794M   1% /run/user/1000

user@ubuntu:~$ lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS

loop0                       7:0    0 55.7M  1 loop /snap/core18/2812

loop1                       7:1    0   55M  1 loop /snap/core18/1880

loop2                       7:2    0 63.9M  1 loop /snap/core20/2105

loop3                       7:3    0 74.1M  1 loop /snap/core22/1033

loop4                       7:4    0 71.3M  1 loop /snap/lxd/16099

loop5                       7:5    0 91.8M  1 loop /snap/lxd/24061

loop6                       7:6    0 40.9M  1 loop /snap/snapd/20290

sda                         8:0    0  200G  0 disk 

├─sda1                      8:1    0    1M  0 part 

├─sda2                      8:2    0    1G  0 part /boot

└─sda3                      8:3    0  199G  0 part 

  └─ubuntu--vg-ubuntu--lv 253:0    0  199G  0 lvm  /

sr0                        11:0    1  4.5G  0 rom  
```