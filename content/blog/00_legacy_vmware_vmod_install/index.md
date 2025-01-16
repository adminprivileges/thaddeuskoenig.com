---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'VMware Vmmod/VMnet Install'
tags:
  - HomeAssistant
  - Web Apps
  - NGINX
  - Legacy
---
After upgrading my Ubuntu Version and subsequently my kernel I tried running  VMware Workstation Pro so I could access my VMs and kept running into the following error code when trying to open the application. Unfortunately, there was no way around it as rebooting, uninstalling/reinstalling, and anything else in my usual low-hanging fruit bag of tricks didn't work so I had to really figure out what was really going on. Lets start with the actual error.  
```
​2022-06-27T12:00:00.000+01:00| vthread-1| W100: Failed to build vmmon.  Failed to execute the build command.

2022-06-27T12:00:00.000+01:00| vthread-1| W100: Failed to build vmnet.  Failed to execute the build command.
```
From here we can see that vmmon and vmnet which are two kernel modules that VMware needs to run are having issues building, this is because the VMware Linux app likes to think its smarter than it actually is,  and its most likely using the wrong package for your kernel which means we can just avoid
this auto check and install it ourselves

## What you actually came for
1. To begin we're going to make sure you have the appropriate packages installed so the rest of this isn't for nothing. This part is Ubuntu-specific, but just swap out apt for whatever package manager fits your distro. 
    ```
    sudo apt install gcc
    sudo apt install build-essentials
    ```
2. Visit this GitHub page and make sure you download the package for your specific version of Vmware Workstation AND kernel version (up until the point release ex. 5.12).
3. From here extract the file contents and enter the directory. 
4. Run the following command to create archives of the modules
    ```
    tar -cvf vmmon.tar vmmon-only
    tar -cvf vmnet.tar vmnet-only
    ```
5. Move the files into your cmware modules folder
sudo mv vmmon.tar vmnet.tar /usr/lib/vmware/modules/source/
6. Install your modules
`sudo vmware-modconfig --console --install-all`
7. All done, go ahead and start VMware Workstation. 