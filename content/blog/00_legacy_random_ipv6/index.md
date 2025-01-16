---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Creating a random private IPv6 Range'
tags:
  - IPv6
  - RFC4193
  - Networking
  - Legacy
---
	
If you're anything like me and (and a good other portion of the internet) you don't want to be bothered with IPv6 addressing because "IPv4 still works" but eventually its time to grow up and realize you need to start becoming more familiar with IPv6 outside of basic textbook knowledge and today I'm gonna teach you how to create a random IPv6 private range for if you need to set up VLANs, VPN tunnels, or whatever. 

First off you're gonna want to check out [RFC4193](https://www.rfc-editor.org/rfc/rfc4193#section-3) or just take my word for it that the IPv6 Local uni-cast prefix (private addressable range) is FC00::/7 and that we're going to want to make one that's unique to us by generating a locally assigned Global ID. The RFC states a good way to do this is by taking a string  (like today's date for example) and combining it with a system specific identifier like your machine ID after that to shake it up, we'll hash it and grab the last 5 bytes. A pretty easy way to do that is with the following one liner: 
```
printf $(echo -n `date +%s%N` && cat /var/lib/dbus/machine-id) | sha1sum | head -c 40 | tail -c 10
```
This will leave you with a 10 digit (5 byte) string that you can append to the fd (or fc) prefix to look like this `fdcc:9a66:f7b2::` because this is an insanely large prefix, best practice is to reduce it to a /64 for simplicity's sake.  