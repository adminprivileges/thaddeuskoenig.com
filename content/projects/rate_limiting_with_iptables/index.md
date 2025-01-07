---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Rate Limiting with IPTables'
tags:
  - IPTables
  - Rate Limiting
  - Linux
  - Firewall
---
## Synopsis
When hosting public servers, whether that be Web, SSH, or whatever you want to roll out, people's biggest concerns are either automated attacks consuming resources and denying your access or those same attacks constantly attempting to authenticate to your application through brute forcing your authentication. I'm going to write this from the perspective of someone simply trying to secure a service that they expect a limited amount of use to, in my case ssh. 

If you ask the internet how to secure your ssh service you'll get a plethora of different answers like changing your default port, only allowing key based authentication, or using tools such as Fail2Ban, and while these are great pieces of advice i just wanted to add another tool to your belt in securing your services whether hosted at home or through a third part VPS. 

Rate limiting is a very simple concept, and effectively tells your server to lower the number of requests that you receive per IP, and block those that exceed your given threshold. I don't typically recommend outright banning IPs as this can be used as a roundabout DoS by an attacker as spoofing an IP address that you would normally come from and getting it banned is a fairly trivial task in today's environment. 

## The Actual Rule
```
/sbin/iptables -N LOGDROP

/sbin/iptables -A LOGDROP -j LOG

/sbin/iptables -A LOGDROP -j DROP

/sbin/iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --set

/sbin/iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent  --update --seconds 60 --hitcount 4 -j LOGDROP
```
## [What the ____does that mean Kobe Bryant?](https://youtu.be/jE_K3NwnOzM)
I'm a fan of logging on pretty much every application, as it allows me to go back in time and see what went wrong when its time to troubleshoot depending on what distribution youre running iptables can log to a couple of different places, but they should be at /var/log/kern.log on Ubuntu since these are kernel generated logs, or /var/log/messages on Red Hat like systems. 

Rate limiting is provided by a module called recent that allows a list of IPs to be cached and used for matchmaking the other switches are for the following: 

- `-tcp` and `--dport 22` are used to specify that these operations are to only be used for ssh (assuming you have it on port 22), you could remove this and have it run for all incoming connections, but i dont recommend it, especially if youre running a web app. 
- `-m state` and `--state NEW` are used to tell iptables that these rules are for new connections only, as if this werent in place, then you run the chance of dropping authenticated ssh connections. 
- `--set` causes the recent module to cache the source ip of the current packet
- `--update` causes the source ip of a packet to be checked against the cached list
- `--seconds 60` and `--hitcount 4` tells makes the rule to be applied against any IP that attempts more than 3 sessions within a minute
- `-j LOGDROP` is set to log and drop packed that match against 4+ packets per second threshold

Additional Note
IPTables rules are not persistent so you have a couple options to ensure these commands get run at boot, but i recommend you turn the above commands into a bash script and either turn it into a [systemd service](https://www.thaddeuskoenig.com/projects/systemd-service-guide) or you make a cronjob out of it to run at reboot, it should look a little like this:
```
@reboot /bin/bash /home/<USER>/ratelimit.sh
```