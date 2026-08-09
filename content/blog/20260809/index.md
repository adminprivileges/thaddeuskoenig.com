---
date: '2026-08-09T15:50:00-05:00'
draft: false
title: '09AUG2026 - Using travel routers the hard way.'
tags:
  - gl.inet
  - travel router
  - pfsense
  - wireguard
  - vpn
  - dns
  - networking
--- 
# 01. Intro
When I travel, I have a few technology essentials that I like to bring along to include 1) travel router 2) streaming device 3) laptop. Im more likely to forget my toothbrush than I am these things and theyre going to be the focal point of this post. I like to bring a travel router for a few different reasons. The primary reason is that I dont trust any shared network so I would like to ensure there is some kind of VPN, either back to my house or to a VPN provider of my choosing. Next is that the devices I bring along with me have the travel router login pre-programmed, so once the router is plugged in they'll auto-connect. Lastly I can continue to use my DNS ad blocking and it just helps me get around any weird network restrictions imposed wherever I am, whether thats having to pay per-device in the captive portal, or blocking websites that I like to go to. I will admit that the way I wanted it set up is very particilar, but for documentation purposed this is what I have going for my situation. 

# 02. Layout
The intended design on my travel setup looks like this 
```
The intended design was roughly this:

                         Internet
                            |
                     WireGuard VPN
                            |
                       GL.iNet Router 
                            | (192.168.8.1)
                 +----------+----------+
                 |                     |
              LAN Clients          Tailscale (100.64.0.0/10)
              (192.168.8.0/24)         |
                              Remote Tailnet (100.110.120.0/24)
                                       |
                                   pfSense (100.110.120.130)----> Unbound/NextDNS
                                       |(192.168.10.1)
                                   remote network (192.168.10.0)
```
It's a bit messy looking but its pretty elegant (in my opinion). When im at a hotel most of my traffic is pushed over the wireguard tunnel to my VPN provider, I highly recommend wireguard in this scenario, because public internet is already slow enough, especially when pushed throgh a VPN, wireguard will give you bit more bandwith than OpenVPN and I think its much easier to set up. I wont outline the wireguard setup here, but if youre planning on rolling out your own VPN and you'd like to learn more, just check out the proceeding links. I still like the DigitalOcean writeup a bit more than the official site: https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-20-04#step-3-creating-a-wireguard-server-configuration with no shortage of documentation on how to set up your travel router as a wireguard client with your favorite VPN provider here https://docs.gl-inet.com/router/en/4/interface_guide/wireguard_client/ the info on getting tailscale setup on these is also here https://docs.gl-inet.com/router/en/4/interface_guide/tailscale/ . The only traffic that is not pushed over the Wireguard connection is the traffic for my tailnet to connect back home. Theres some services at my house that I like to maintain the ability to connect to, such as my DNS server, and while thats easy enough on my phone it's not as easy on a streaming device. Back home I have a pfsense firewall that not only deals with local DNS records for my domain, but also serves those routes to my tailnet. 

The final goal was therefore:
```
Internet traffic
Client -> GL.iNet -> WireGuard VPN -> Internet

Internal DNS
Client -> GL.iNet -> Tailscale -> pfSense/Unbound

Remote private network
Client -> GL.iNet -> Tailscale -> Subnet Router -> Remote Host
```

# 03. Diagnosing DNS
From this pont on im assuming you have your travel router connected as a wireguard client and it is actively on your tailnet. With this setup, I was having issues with DNS (shocker). The main issue is that I was able to resolve whatever IP I wanted via my firewall on the router itself when i SSHed to it, but not when connected to a client device, I could resolve public IPs (kinda) but not private DNS records on my server, which breaks my ability to use my reverse proxies. I saw if I specified the IP address of the firewall in my nslookup (nslookup example.thaddeuskoenig.com 192.168.10.1) it worked. Simplified, my current predicament was this. 
```
GL.iNet -> pfsense (local)                    WORKS
Client  -> pfsense (local)                    WORKS
GL.iNet -> pfsense (tailscale)                WORKS
Client  -> pfsense (tailscale)                FAILS
```
This kinda clued me into the fact that there was something up with  the DNS forwarding on the travel router.

## Manual DNS vs. DNS Proxy on GL.iNet
Manual DNS is pretty usedul in some cases. The GL-iNet even has documentation on how to ensure its working through a Wireguard connection here: https://docs.gl-inet.com/router/en/4/tutorials/route_vpn_client_dns_to_server_upstream_dns/ . My problem is that my preferred DNS server was not on the other side of the wireguard connection (i know, technically it was since tailscale is just wireguard). It was that my DNS server was part of my tailnet and I wanted to connect to it that way, the documentation specifically calls out proxy mode for when youre using a local DNS server, so I went with that method: https://docs.gl-inet.com/router/en/4/interface_guide/dns/#dns-proxy . This setting can be found in the web portal via `Network -> DNS -> DNS Server Settings -> DNS Proxy` from here I could do normal nslookups on my client devices without having to specify a DNS server. The remaining problem here was that it worked for the internal pfsense IP (192.168.10.1), but not the tailscale interface for it (00.110.120.130). 

*Note: Its also a good idea to turn off DNS rebinding protction. With this on, you may have issues connecting to things in your remote network. This setting is fine for the average person, but if you have services hosted in your home network that you reach via private IP, then you should disable this setting*

## Fixing pfSense DNS Resolution
My revelation from before turned into a pretty big clue for what I needed to do next. My pfsense was reachable through Tailscale, but Unbound did not appear to be working on that interface. pfSense uses Unbound for its DNS Resolver. Netgate documents that the Network Interfaces setting controls which interfaces Unbound binds to, and that when specific interfaces are selected, Unbound only listens on those selected interfaces. Queries sent to other firewall addresses are silently discarded. An easy solition for this is to go into your pfSense and select ` Services -> DNS Resolver -> General -> Network Interfaces: All ` I didnt want to go this route, because that would potentially cause Unbound to listen on interfaces that I didnt want it listening on like WAN. 
### Checking what UnBound is listening on
Its a good idea to take note of your pfsense tailscale IP. If youre SSHed to the router you can run an `ifconfig tailscale0`. From there its a good idea to run to see what interfaces youre actually listening on `sockstat -4 -l | grep ':53'`. If you run this command and dont see your tailscale IP, your unbound is not resolving because its not listening on that interface. You can also `grep -n 'interface:' /var/unbound/unbound.conf` to check out your listening interfaces from your unbound config. If your Ip is not showing, you can simply add it explicityly in the web interface, go to `Services -> DNS Resolver -> General Settings -> Custom Options` and then add the following, with the address of YOUR tailscale interface for your firewall.
```
server: 
  interface: 100.110.120.130 #CHANGE THIS
```
If youre also running NextDNS, you can repeat the server line making your full custom options field look more like this
```
server:
    interface: 100.110.120.130

forward-zone:
    name: "."
    forward-tls-upstream: yes
    forward-addr: 45.90.28.0#CONFIG_ID.dns1.nextdns.io
    forward-addr: 2a07:a8c0::#CONFIG_ID.dns1.nextdns.io
    forward-addr: 45.90.30.0#CONFIG_ID.dns2.nextdns.io
    forward-addr: 2a07:a8c1::#CONFIG_ID.dns2.nextdns.io
```
 
### Access Lists
Theres one last thing you need to do in pfSense for this step and thats to ensure that you include a custom Access List for tailscale device resolutons. You can use a single device, multiple, or just the whole tailscale IP range. Whaterver you want. To do this, youre gonna go to ` Services -> DNS Resolver -> Access Lists -> Access Lists -> Add` and create an entry for a single address, multiple networks, or `100.64.0.0/10` for all of tailscale. 

# 04. Fix Routing Precedence
To this point DNS is working, but when youre connected to both the wireguard client and the tailnet, the wireguard gateway will take priority for most things outside of DNS resolution. This breaks the ability to connect to our devices directly in most cases because the other side of your VPN connection will likely just throw out a private IP when its pushed over that pipe.  
To figure out whats going on, a good first place to check is your routing. On the GL-iNet router run the following on your gl-inet router to see your shares subnets. 
```
ip route show table 52
```
Youll see a bunch of devices on your tailnet, but most importantly for now you should see this
```
192.168.10.0/24 dev tailscale0 
```
If you do not see this, make sure you have ` Allow Remote Access WAN` on in your gl-inet tailscale settings. 
If you do see this, and its still not working, the explanation is (relatively) simple: Linux policy routing adds another layer on top of your routing table, and the routing policy is where the "magic" of tailscale lives. while on your machine, run
```
ip rule show
```
You should see something like this:
```
0: from all lookup local 
48: from all to 172.18.0.0/16 lookup main 
49: from all to 192.168.8.0/24 lookup main 
50: from all to 100.100.100.100 lookup 52 
1099: from all fwmark 0x80000/0xc0000 lookup main 
1100: from all lookup main suppress_prefixlength 0 
1101: not from all fwmark 0x8000/0xc000 lookup 8000 
5210: from all fwmark 0x80000/0xff0000 lookup main 
5230: from all fwmark 0x80000/0xff0000 lookup default 
5250: from all fwmark 0x80000/0xff0000 unreachable 
5270: from all lookup 52 
32766: from all lookup main 
32767: from all lookup default
```
Out biggest problem lies here:
```
1101: not from all fwmark 0x8000/0xc000 lookup 8000
5270: from all lookup 52
```
Table `8000` is part of the wireguard policy and `52` is tailscale. Because 1101 is much lower than 5270, th wireguard policy is checked much before tailscale. Normally this means that we shouldnt have even been able to use DNS, but the auto created `50: from all to 100.100.100.100 lookup 52` rule allows out resolution to go through first. 

We could simply change the order of precedence here, but thats not a good idea. Tailscale is a relatively complicated piece of software and messing with its networking in that way can cause issues, especially in weird setups like mine, so I opted for the solution of including a few routing rules before the firewall rule to solve this issue. To do so, you simply use the ip command to add new rules. 
```
ip rule add pref 1000 to 192.168.10.0/24 lookup 52
```
You can also add a rule for tailscale IPs to ensure those are routed correctly
```
ip rule add pref 990 to 100.64.0.0/10 lookup 52
```
This is a very simple fix, but the only issue is these commands will not persist a reboot, which takes us to our next step. 

## Making rules persistent
Because these rules dont persist a reboot, we are going to use a simple helper script that will run at boot and ensure our routing precedence is adhered to. On your router `vi /usr/bin/tailscale-route-exceptions` and paste the following ** and make sure to change the IPS at the end**
```
#!/bin/sh

add_rule() {
    PREF="$1"
    SUBNET="$2"

    ip -4 rule show | grep -q "^${PREF}:.*to ${SUBNET} .*lookup 52" || \
        ip -4 rule add pref "$PREF" to "$SUBNET" lookup 52
}

# Direct Tailscale IPv4 addresses
add_rule 990 100.64.0.0/10

# Tailscale-advertised remote networks
add_rule 1000 192.168.10.0/24 #CHANGE ME

exit 0
```
Make the script executable with `chmod +x /usr/bin/tailscale-route-exceptions`, run it and then check your work witj `ip rule show`.

To make this actually run at boot, we're going to need to use the hotplug system, so create a file with `/etc/hotplug.d/iface/99-tailscale-route-exceptions` . And paste the following
```
#!/bin/sh

case "$ACTION" in
    ifup|ifupdate)
        (
            sleep 3
            /usr/bin/tailscale-route-exceptions
        ) &
        ;;
esac
```
and then ensure to `chmod +x /etc/hotplug.d/iface/99-tailscale-route-exceptions`

Lastly, since i enjoy a good belt and suspenders approach, we'll `vi /etc/rc.local` and add `/usr/bin/tailscale-route-exceptions` **on a line BEFORE exit 0**

# Conclution

From here you should be done, I will revisit this blog post to add some extra troubleshooting steps, and reading another time, but I spent too much time troubleshooting and writing this as it is so I need a break.
