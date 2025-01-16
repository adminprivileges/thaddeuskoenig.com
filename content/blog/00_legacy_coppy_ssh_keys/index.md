---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Copy ssh keys without ssh-copy-id'
tags:
  - SSH
  - Keys
  - PKI
  - Legacy
---
I ran into this issue today so I thought that i would put a solution that I found on here. If you ever run into a situation in which you need to copy your ssh keys to another box but you dont have the handy dandy ssh-copy-id tool, the following one liner should work. Check out this oracle web page for more info.
```
cat ~/.ssh/id_rsa.pub | ssh <USER>@<IP> 'cat >> .ssh/authorized_keys && echo "Done"'
```
