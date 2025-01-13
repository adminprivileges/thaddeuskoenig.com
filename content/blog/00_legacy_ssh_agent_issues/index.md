---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'SSH Auth Agent issues on FreeBSD'
tags:
  - SSH
  - SSH Agent
  - FreeBSD
---
So i was trying to make a script that would automate SSL key exports from a BSD machine to another so I don't have to go through the pain of doing it manually every time and i ran into the issue of me not being able to ssh-copy-id my files over but i keep running  into the issue of it not being able to connect to my auth agent due to a lack of keys even though i generated them with ssh-keygen, nonetheless no matter what I did i kept getting met with: 
```
Could not open a connection to your authentication agent.

no keys found
```
The fix to this is pretty simple. What i found is that i was being spoiled by Linux so i never really had to add keys to my ssh-agent, but  you may have to start with getting your ssh-agent right using 
```
eval `ssh-agent -c`
```
Which will generate C-shell commands on stdout.  Next you just simply add your key to the agent using 
```
ssh-add ~/.ssh/id_rsa
```