---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'ProxMox LXC Deployment'
tags:
  - Proxmox
  - Containers
  - Ansible
  - LXC
---
In an attempt to automate the deployment as well as get my feet wet with a little Ansible in my homelab i created a playbook to automate the deployment of the minimal LXCs that build to deploy certain apps like pi-hole, nginx proxies, and sql databases. The documentation on certain things were pretty sparse so I will try to explain the intentions that I had, the issues that I ran into, as well as why I chose the routes that I took for my solution. 

## Installing Ansible
The instructions for installing ansible are pretty straightforward. Some application repositories have the package archived in order for you to do something like a sudo apt install ansible but I don't really recommend it. The python module works great. See the official installation instructions here: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
```
python3 -m pip install --user ansible
```
This will get you all of the packages you need to get ansible running, unfortunately, if you're on proxmox and use the default root user to run most of your commands, then you won't have the appropriate path to run these commands without an absolute path so trying to verify the install with ansible --version will fail.  To fix this, just add this line in your .bashrc to fix your path. 
```
export PATH=$PATH:/root/.local/bin/
```
From there you can either
- logout and log back in
- run source ~/.bashrc
- run `bash` (this will start a new bash session within your current session)

### Installing the ProxMox Ansible packages
The proxmox ansible packages reside in the community.general repo, so we must first install that

ansible-galaxy collection install community.general

Now we just need to verify we just need to install the proxmoxer package from python

sudo pip3 install proxmoxer

## Running the playbook
Its easiest to go through the playbook sequentially and explain why each of the parts are added.

To begin I wanted to create a way to allow for user input to set variables such as the name of the container. The Container_Name variable allows for unique names for the hostname variable (explained later) allows you to leave out the vmid variable and allow proxmox to simply assign the next available id to your container. I also wanted the playbook modular for testing, these other prompts can be taken out and replaced with static values. For more info on prompts check out the official documentation here: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_prompts.html
Some things of note
- My ProxMox password had some special characters that either ansible or yaml didn't like, but thankfully the unsafe variable allowed for it to work. In the future, I'm going to move all the secrets to an ansible vault, but this was the easiest thing to do while I was trying to figure out everything else.
```
  vars_prompt: 
    - name: 'ProxMox_Hostname'
      prompt: "Please input ProxMox hostname" 
    - name: 'ProxMox_Password'
      prompt: "Please input ProxMox password" 
      private: true #This hides the password
      unsafe: true #This stops special chars from breaking ansible
    - name: 'CT_Password'
      prompt: "Please input container password" 
      private: true #This hides the password
      unsafe: true #This stops special chars from breaking ansible
    - name: 'Container_Name'
      prompt: "Enter a unique name for your containers"
      private: false
```
Here we actually create the container. The official documentation was great to get a start, but I had to make a few changes in order for this to be actually usable in my environment such as
- **Hosname:** The official documentation shows you how to create singular containers via passing the vmid itself as an argument. This is cool and all, but I didn't want to have to remember what the last vmid was. Its a lot easier to do a unique hostname and let proxmox figure that out.
- **OSTemplate:** local:vztmpl/ is simply the location in which ct templates are stored on the local machine by default
- **Storage:** because local is the default argument for containers and I am in a ZFS environment, i needed to specify the default zfs location here with local-zfs 
- **netif:** The documentation recommends passing this as a JSON list, which is ugly but works fine although in order to get it to actually work, I had to remove ipv6=dhcp and add firewall=1 i like doing static IP assignments at the network level so clients are always given their static addresses that way. 
- **Hookscript:** I will explain this later, but this tells proxmox to perform certain actions based on this container's lifecycle
```
  - name: 'Create Container'
    community.general.proxmox:
      api_user: 'root@pam' # Proxmox user
      api_password: '{{ ProxMox_Password }}'
      api_host: '{{ ProxMox_Hostname }}' # Proxmox hostname
      password: '{{ CT_Password }}' #Container password
      hostname: '{{ Container_Name }}' # Container hostname
      node: 'pve01' # Name of Proxmox host
      ostemplate: 'local:vztmpl/debian-11-standard_11.6-1_amd64.tar.zst'
      disk: '16'      
      cpus: '1'
      cores: '1'
      memory: '512'
      storage: local-zfs
      netif: '{"net0":"name=eth0,firewall=1,ip=dhcp,bridge=vmbr0"}' #IPV6 needs to be left out and firewall enabled in order to acess the ct
      hookscript: 'local:snippets/install_ssh.sh' #This is a script that SHOULD run at start but hookscripts are largely undocumented
      unprivileged: true
      description: 'created with ansible'
      features:
       - nesting=1
```
This just waits long enough for the container to be initialized and assigned a hostname. Without this wait, the start command will fail because it tries to reference a container by a hostname that doesn't exist yet. 
```
  - name: Pause for 10 seconds for container to finish build
    ansible.builtin.pause:
      seconds: 10 #Just enough time for the vm to register a hostname to be referenced in start
```
This is simple enough. Nothing Fancy. It just starts the container because you can't create and run them in the same task. 
```
  - name: Start container
    community.general.proxmox:
      api_user: root@pam
      api_password: '{{ ProxMox_Password }}'
      api_host: '{{ ProxMox_Hostname }}'
      hostname: '{{ Container_Name }}' # Container hostname
      state: started
```
The Whole Playbook.
```
---
- name: Create new LXC container

  hosts: localhost
  vars_prompt: 
    - name: 'ProxMox_Hostname'
      prompt: "Please input ProxMox hostname" 
    - name: 'ProxMox_Password'
      prompt: "Please input ProxMox password" 
      private: true #This hides the password
      unsafe: true #This stops special chars from breaking ansible
    - name: 'CT_Password'
      prompt: "Please input container password" 
      private: true #This hides the password
      unsafe: true #This stops special chars from breaking ansible
    - name: 'Container_Name'
      prompt: "Enter a unique name for your containers"
      private: false
  tasks:

  - name: 'Create Container'
    community.general.proxmox:
      api_user: 'root@pam' # Proxmox user
      api_password: '{{ ProxMox_Password }}'
      api_host: '{{ ProxMox_Hostname }}' # Proxmox hostname
      password: '{{ CT_Password }}' #Container password
      hostname: '{{ Container_Name }}' # Container hostname
      node: 'pve01' # Name of Proxmox host
      ostemplate: 'local:vztmpl/debian-11-standard_11.6-1_amd64.tar.zst'
      disk: '16'      
      cpus: '1'
      cores: '1'
      memory: '512'
      storage: local-zfs
      netif: '{"net0":"name=eth0,firewall=1,ip=dhcp,bridge=vmbr0"}' #IPV6 needs to be left out and firewall enabled in order to acess the ct
      hookscript: 'local:snippets/install_ssh.sh' #This is a script that SHOULD run at start but hookscripts are largely undocumented
      unprivileged: true
      description: 'created with ansible'
      features:
       - nesting=1

  - name: Pause for 10 seconds for container to finish build
    ansible.builtin.pause:
      seconds: 10 #Just enough time for the vm to register a hostname to be referenced in start

  - name: Start container
    community.general.proxmox:
      api_user: root@pam
      api_password: '{{ ProxMox_Password }}'
      api_host: '{{ ProxMox_Hostname }}'
      hostname: '{{ Container_Name }}' # Container hostname
      state: started
```
## Hookscripts
So hookscripts are largely undocumented which was a source of frustration, but through some trial and error, i was able to land on a solution that works for me (for now). When you attach a hookscript to a container it tells proxmox to run certain commands on the host at different points in the containers lifecycle they are: 
- pre-start
- post-start
- pre-stop
- post-stop

The only thing I'm concerned with for my use case is the post-start so that I can get my VM to a point in which it can be accessed by SSH. I took advantage of the example script provided within proxmox and moved it to /var/lib/vz/snippets which is a standard location for scripts from there i just added the lines in bold. There's probably a better way to do it but again, the documentation is pretty sparse, so I just used the perl system function to run the commands I need. 
```
#!/usr/bin/perl
# Exmple hook script for PVE guests (hookscript config option)
# You can set this via pct/qm with
# pct set <vmid> -hookscript <volume-id>
# qm set <vmid> -hookscript <volume-id>
# where <volume-id> has to be an executable file in the snippets folder
# of any storage with directories e.g.:
# qm set 100 -hookscript local:snippets/hookscript.pl
use strict;
use warnings;
print "GUEST HOOK: " . join(' ', @ARGV). "\n";

# First argument is the vmid
my $vmid = shift;

# Second argument is the phase
my $phase = shift;



if ($phase eq 'pre-start') {

    # First phase 'pre-start' will be executed before the guest
    # is started. Exiting with a code != 0 will abort the start
    print "$vmid is starting, doing preparations.\n";

    # print "preparations failed, aborting."
    # exit(1);

} elsif ($phase eq 'post-start') {

    # Second phase 'post-start' will be executed after the guest
    # successfully started
    # Push the script to the container.
    system("/usr/sbin/pct push --perms 700 $vmid /var/lib/vz/snippets/install_ssh.sh /root/install_ssh.sh");

    #Run the script
    system("/usr/sbin/pct exec $vmid /root/install_ssh.sh");
    #Delete the script after running
    system("/usr/sbin/pct set $vmid --delete hookscript");
    print "$vmid started successfully.\n";

} elsif ($phase eq 'pre-stop') {

    # Third phase 'pre-stop' will be executed before stopping the guest
    # via the API. Will not be executed if the guest is stopped from
    # within e.g., with a 'poweroff'

    print "$vmid will be stopped.\n";

} elsif ($phase eq 'post-stop') {

    # Last phase 'post-stop' will be executed after the guest stopped.
    # This should even be executed in case the guest crashes or stopped
    # unexpectedly.

    print "$vmid stopped. Doing cleanup.\n";

} else {
    die "got unknown phase '$phase'\n";
}

exit(0);
```
The script simply pushes a startup script to the container, runs it, and "unhooks" the script so this isn't run everytime the container restarts. Doing this via a hookscript is valuable because the ID of the vm is passed by proxmox in the $vmid variable making it easy to target the container that launched the script. The script that is pushed to the host simply enables password authentication for the root account to the container and starts ssh, after that it creates a file as proof that it ran.
```
#!/bin/bash

#I can probaly use the $VM_ID variable and run the commands from the host via pct exec. maybe the play if to get the perl script to run pct exec with the vm_id paramater and pass this script

sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

systemctl enable --now ssh

touch /root/.hookscript_sucessful
```