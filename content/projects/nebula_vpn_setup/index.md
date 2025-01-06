---
date: '2024-01-04T12:00:00-05:00'
draft: false
title: 'Nebula VPN Setup'
tags:
  - Mesh Networking
  - VPN
  - Nebula
  - Certificates
---

Most people see VPN and they think of the commercial, hide myself within another million people’s traffic, product such as Nord or PIA. But this article is simply an interesting take on the technology used to bridge the physical gap between work-spaces that many professionals both technical and not became intimately familiar with over the recent 2021 pandemic.
## What is Nebula and how is it different?

In a traditional sense a VPN functions as a router that bridges different LANs over an encrypted tunnel, and the traffic passed between LANs travels much like traditional traffic would when passed between LANs, in a star-like topology where all traffic destined for a peer must first pass through a switching/routing device, but what if that wasn't the case? What if you could simply cut out the middle man and pass traffic directly between networked nodes in a VPN network? That’s exactly what Nebula brings to the table.
## Okay, how does it do that?

When using nebula a user sets up a “Lighthouse”, which much like the object of its namesake serves the purpose of aiding in the navigation of lost devices. If two devices have a logically “closer” route to on another than through the lighthouse, they can simply poll the lighthouse for the IP and UDP source port of on another and erect a new tunnel directly to one another by spoofing the lighthouse IP as the source which effectively can allow two devices to communicate directly to one another even if both are behind NAT. If this connection fails for any reason the devices can call back and route traffic through the lighthouse much like a traditional VPN.
## Considerations

Before we get into setup, I want to point out a few things of note. Much like Openvpn and Wireguard, nebula uses UDP, which is a connectionless protocol, which allows the flexibility to make this technology viable. That being said this can cause some problems if certain security protocols are in place such as Deep Application Boundary Validation which would in turn notice the disingenuous nature of the spoofed packets and drop them before they reach the target. Nebula is resilient enough to still function if this is happening on one side, but id both users are subject to such scrutiny then Nebula is completely off the table. Also you're going to want a lighthouse that always maintains a static IP address, the clients can have dynamic IP addresses, but the second the lighthouse IP changes you're going to have a lot of headaches as the lighthouse public IP is hard-coded into the configuration files resident on all the devices, the cheapest and most popular way to achieve this is by simply renting a cloud VPS from your favorite provider. I typically use AWS, but Linode and Digital Ocean sponsor a lot of my favorite podcasts and Youtube Channels so i know they're cheaper, and have many codes that let you try it out on their dime for a couple months (This link will give you a $100 credit courtesy of my favorite podcast 2.5 Admins).
## How do i set it up?

1. Navigate to the Releases section of the Nebula Github Repo, and grab the tarball thats appropriate for your architecture from the assets (Today I will be using cloud VPS nodes from A Cloud Guru).
    - If youre using linux you can just wget the link, the following code will pull the 1.0.4 linux 64 bit tarball
```
wget https://github.com/slackhq/nebula/releases/download/v1.4.0/nebula-linux-amd64.tar.gz
```
2. From there make sure to unzip and extract the files which will result in two executables named “nebula” and “nebula-cert”.
```
gunzip nebula-linux-amd64.tar.gz
tar xvf nebula-linux-amd64.tar.gz
```
3. Create your own certificate authority by using the provided executable.
```
./nebula-cert ca -name "Thaddeus Nebula Network"
```
4. Now we will use the “nebula-cert” executable to create keys and certs for the lighthouse and clients to authenticate to one another (note, you will also dictate your IP scheme here, try to use something different that wont conflict with your home network, or anyone else’s).
```
./nebula-cert sign -name "lighthouse" -ip "192.168.111.1/24"
./nebula-cert sign -name "client1" -ip "192.168.111.2/24"
./nebula-cert sign -name "client2" -ip "192.168.111.3/24"
```
5. Here you will have a .crt and a .key file for lighthouse, and also client one and two, create a /etc/nebula directory on your lighthouse and move “lighthouse.key”, “lighthouse.crt” and “ca.crt” to the directory, then proceed to scp the other files to their respective machines and directories. Every machine should have a copy of the ca.crt file, but the ca.key file should be kept somewhere safe. For then new clients need to be added to the network. If you would like they it can be removed from the lighthouse completely and keys can be generated from your own workstation.
6. From each machine make sure to download the nebula tarball and extract the contents to a directory of your choosing.

7. From here we will download the Nebula example configuration file and make a few tweaks.
```
#1. Change the names of these files to match those that you set in the earlier step. 
pki:                        
   ca: /etc/nebula/ca.crt                       
   cert: /etc/nebula/lighthouse.crt                       
   key: /etc/nebula/lighthouse.key

#2. Input any static hosts that you would like saved here (set the lighthouse IP at the very least)
static_host_map:              
 "192.168.32.1": ["172.31.117.86:2222"]

#3. Make sure this is set true on your lighthouse config
lighthouse:
 am_lighthouse: true

#4. Make sure this is empty on the lighthouse, but filled on the clients. 
hosts
 - 

#5. This is where you will set the port you would like the lighthouse to listen on. 
listen:
 host: 0.0.0.0
 port: 2222

#6. For testing purposes I opened my firewall completely, make sure to tune this for your specific purposes
 outbound:
   # Allow all outbound traffic from this node
   - port: any
     proto: any
     host: anyinbound:
   # Allow all traffic between any nebula hosts
   - port: any
     proto: any
     host: any
```
8. Once you’ve set up configuration files between the devices you in your nebula network, you can start up your light house using the “nebula” executable.
```
sudo ./nebula -config ./config.yml
```
9. The setup is typically done with in seconds and your output should look a bit like this.
```
INFO[0000] Firewall rule added                        
firewallRule="map[caName: caSha: direction:outgoing endPort:0 groups:[] host:any ip: proto:0 startPort:0]"
INFO[0000] Firewall rule added                        
firewallRule="map[caName: caSha: direction:incoming endPort:0 groups:[] host:any ip: proto:0 startPort:0]"
INFO[0000] Firewall started                     firewallHash=21716b47a7a140e448077fe66c31b4b42f232e996818d7dd1c6c4991e066dbdb
INFO[0000] Main HostMap created                      
network=192.168.32.1/24 preferredRanges="[]"
INFO[0000] UDP hole punching enabled                   
INFO[0000] Nebula interface is active                    
build=1.4.0 interface=nebula1 network=192.168.32.1/24 udpAddr="0.0.0.0:2222"
```
10. Issue the same command on you client device, if everything is correct you will see a successful handshake in your output
```
INFO[0000] Firewall rule added                           firewallRule="map[caName: caSha: direction:outgoing endPort:0 groups:[] host:any ip: proto:0 startPort:0]"
INFO[0000] Firewall rule added                           firewallRule="map[caName: caSha: direction:incoming endPort:0 groups:[] host:any ip: proto:0 startPort:0]"
INFO[0000] Firewall started                          firewallHash=21716b47a7a140e448077fe66c31b4b42f232e996818d7dd1c6c4991e066dbdb
INFO[0000] Main HostMap created                          network=192.168.32.3/24 preferredRanges="[]"
INFO[0000] UDP hole punching enabled                   
INFO[0000] Nebula interface is active                    build=1.4.0 interface=nebula1 network=192.168.32.3/24 udpAddr="0.0.0.0:2222"
INFO[0000] Handshake message sent                        handshake="map[stage:1 style:ix_psk0]" initiatorIndex=69986371 udpAddrs="[172.31.117.86:2222]" vpnIp=192.168.32.1
INFO[0000] Handshake message received                    certName=lighthouse durationNs=21471018 fingerprint=7c894901bde2033f3a942941c899f4de1290018ffd9e998e856efa4186e2987a handshake="map[stage:2 style:ix_psk0]" initiatorIndex=69986371 remoteIndex=69986371 responderIndex=831563762 sentCachedPackets=1 udpAddr="172.31.117.86:2222" vpnIp=192.168.32.1
```
11. To take full advantage of the mesh nature of the technology set this up on another client. If they are logically closer to one another, you can start a packet capture on your lighthouse and see that it is being completely subverted in the transference of packets between nodes.