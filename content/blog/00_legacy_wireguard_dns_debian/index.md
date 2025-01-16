---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Wireguard DNS issues with Debian'
tags:
  - Wireguard
  - VPN
  - Debian
  - Legacy
---
	
When utilizing debian based distros, I've run into this issue a lot when I'm trying to use wireguard and specify DNS settings:
```
/usr/bin/wg-quick: line 31: resolvconf: command not found
```
 A lot of this is because of systemd has its own resolvconf so you can simply make a symlink to get wireguard running normally:
```
sudo ln -s /usr/bin/resolvectl /usr/local/bin/resolvconf 
```