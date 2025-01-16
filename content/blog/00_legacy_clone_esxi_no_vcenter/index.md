---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'How to Clone a Virtual Machine in ESXI (without vCenter)'
tags:
  - Vmware
  - ESXI
  - Virtual Machines
  - vCenter
  - Legacy
---
	
When you're running ESXI without vCenter, sometimes really niche things come up and you have situations that are kind of a pain without the appropriate management software so sometimes you gotta be a little janky. 

- So Primarily you're going to want to power off the VM you want to clone.

- Then you want to right click the VM and chose Edit Settings.

  - In the Virtual Hardware tab take not of the name and location of your Disk File.

- Next you're going to open your Datastore Browser by going Storage and in the Datastores tab click the Datastore Browser button. 

- Now that youre in the datastore browser create a directory at your prefered location with the name of the VM you would like.

- Now youre going to go to the location of your original VM and copy the .vmdk and .vmx files into your new directory.

  - Sometimes this can take a while, if you want to monitor the process. Click the Monior sidebar, then the Tasks tab. 

- When your files are moved over, you can rename them if youd like.

- Finally, right click the .vmx file and choose Register VM.  

- Rename the actual VM if youd like. 