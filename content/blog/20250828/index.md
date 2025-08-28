---
date: '2025-08-28T12:47:00-05:00'
draft: false
title: '28AUG25 - Dont Forget to update initramfs'
tags:
  - zfs
  - zfsbootmenu
  - initramfs
  - boot
---
## 01. Intro
I use ZFSBootMenu to manage my boot environments and help me roll back when I inevitably do something destructive to my system. Today I decided to change my encryption password because my current one has a key combination that confused my fingers so I almost always typed it incorrectly the first time. I changed my zfs rpool password, but I neglected to update initramfs. Im documenting the steps to properly change my password here so that i dont forget.

1. Find out where your key is stored using  zfs
    ```
    zfs get keylocation rpool
    ```
2. Edit the password keyfile using your new password.
    ```
    sudo vim /etc/zfs/rpool.key
    ```
3. Update Password using the utility 
    ```
    sudo zfs change-key -o keyformat=passphrase rpool
    ```
4. Update initramfs.
    ```
    sudo update-initramfs -u
    ```
The third step is the crucial step that I completely neglected. If this step is missed then initramfs will still be built with the old password in mind and fail to build upon next boot. The only way to fix it at that point is to mount a live filesystem and chroot into your environment to update initramfs. Thankfully zfsbootmenu has a live chroot environment built in, so if you're like me and you do this you can enter the zfsbootmenu menu, chose the chroot option on the appropriate dataset and fix yourself. Hopefully i come back to reference this the next time.