---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Proxmox Qcow2 Import'
tags:
  - Proxmox
  - qcow2
  - qemu
  - Legacy
---
## Introduction
For those who aren't familiar with the concept. Different hypervisors use different file formats to represent VM images and due to the fact that Proxmox uses Qemu (Quick Emulator)/KVM (Kernel-based Virtual Machine) to provide virtualization, it in turn uses Qcow2 (Qemu Copy-On-Write) as the storage file format for virtual machines data. The process to move Qcow2 files into Proxmox may not be as straight forward as it is on VMware, but its still a relatively painless process. 

## Environment Setup
First, you're going to want to need a place to store the Qcow2 files that you're going to copy and virtualize. Its probably best to put them near where VM ISOs and container images are stored like such:
```
mkdir /var/lib/vz/template/qcow2
```
After this is created you can move the qcow2 file to your proxmox instance however you see fit. If you don't have it saved quite yet, it may be easiest to simply use wget to pull the image from whatever repo you're hoping to grab it from so save the bandwidth of saving it to your disk and then moving it to Proxmox. 

## Creating the VM
1. We will begin setup by going to the Create VM action.

2. From here in the General tab, we can give our VM a name, it should be automatically assigned an unique VM ID that you will need to notate for later.

3. In the OS tab, ensure that you click the Do not use any media radio button, from there you can either leave the guest OS as the default Linux, or change it to Other to avoid any possible default configuration issues.

4. In the System tab we should be able to leave all of the defaults here.

5. In the Disks tab we should change our VM's size to how large we want it to be in Disk Size. You can change the Format to Qemu Image Format but the default Raw format should work, my Proxmox wouldn't give me the option to change from raw. 

6. In the CPU tab, give your VM the compute resources that it needs.

7. In the Memory tab, give your VM the memory that it needs. 

8. In the Networking tab, ensure your VM has the networking capabilities that it requires if you would like it to connect over the network. 

9. From there you can hit the Finish button to create your VM.

## Importing Disks
Up to this point, we've simply created a VM without a bootable operating system. From here we will actually import the Qcow2 image and therefore make our VM usable. Ensure you have these pieces of information handy: VM ID, Proxmox Storage Cluster Name(usually just local or local-zfs), absolute file path of the QCOW2 image. 

From your terminal, importing the image is fairly easy using the built-in command: 
```
qm importdisk <VM_ID> <QCOW2_FILE> <PROXMOX_ STORAGE>
```
After that, watch the transfer fly. 

## Swapping Disks
1. Click on your VM and then go to Hardware > Click the Unused Disk at the bottom of the hardware array > and then click the Edit option at the top. 

2. From there we will change the Bus Device  to VirtIO Block  to ensure our VM can take advantage of the best possible performance. 

3. Now we need to change the boot order, so go to Options > Boot Order and move the VirtIO disk to the top. Click OK to save. 

From here you should be able to boot your VM. Give yourself a pat on the back. 