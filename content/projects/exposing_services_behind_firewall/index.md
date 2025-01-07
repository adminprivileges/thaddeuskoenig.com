---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Exposing Services Behind a Firewall'
tags:
  - Firewall
  - Wireguard
  - IPtables
  - Linux
---
## Synopsis
Sometimes you may find yourself in a situation in which you would like to host a service but sometimes you're stuck behind a double NAT, or even just a firewall in which you do not control all of the rules for. Here I'm going to show you how to use Wireguard VPN to finagle around this issue and simply give your private server a public IP address. Important note: in order for this to work, you will need a publicly accessible VPS, preferably with a static IP address, i would recommend either AWS Lighsail or Linode as they are both cost-effective options with their cheapest tiers being bout $3.50/mo and $5/mo respectively (I like Linode's speed more, they aren't paying me to say this although I wish they were).

## Initial Setup
First off, you're going to want to ensure that you have wireguard installed on both ends, thankfully wireguard is upstream in the linux kernel now so you should be able to install it with the package manager of your choice, here I'm going to use Ubuntu server on both ends since it's pretty ubiquitous, although most of these commands will be the same across operating systems with the exception of package installs. 
```
sudo apt install wireguard wireguard-tools
```
Run the following command on both ends to generate your public and private keys
```
wg genkey | tee privatekey | wg pubkey > publickey
```
From here you're going to create files names /etc/wireguard/wg0.conf on both ends that look like the following:
**VPS**
```
[Interface]
PrivateKey = <privatekey>
ListenPort = <UDP_Port> (ex. 51820)
Address = <Desired_Gateway_IP_ADDR> (ex. 192.168.67.1)

[Peer]
PublicKey =  <Public_Key_of_HomeServ>
AllowedIPs = <Desired_HomeServ_IP> (ex. 192.168.67.2/32)
```
** Home Server:** 
```
[Interface]
PrivateKey = <privatekey>
Address = <Desired_HomeServ_IP>  (ex. 192.168.67.2)

[Peer]
PublicKey =  <Public_Key_of_VPS>
Endpoint = <Public_Socket_of_VPS> (ex. 1.2.3.4:51820)
AllowedIPs = <Private_IP_of_VPS>(ex. 192.168.67.1/32)
PersistentKeepalive = 25
Start and enable wireguard on both machines (this will start it on every boot as well).
sudo systemctl enable --now wg-quick@wg0
```
Try a Quick ping on either side to ensure connectivity is good. Then we're going to start up a netcat listener on your home server to simulate a service being run with the following command if you already have a service, skip this step but figure out what port it listens on. 
```
netcat -l 2200
```
On your VPS were going  to need to run the following command to grab a couple network parameters that will be important for Iptables rules (mainly your interface name). 
```
sudo ip -4 addr show scope global
```
Here we're gong to set up some IPtables rules on the VPS
```
sudo iptables -P FORWARD DROP

sudo iptables -A FORWARD -i eth0 -o wg0 -p tcp --syn --dport 2200 -m conntrack --ctstate NEW -j ACCEPT

sudo iptables -A FORWARD -i eth0 -o wg0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

sudo iptables -A FORWARD -i wg0 -o eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 2200 -j DNAT --to-destination 192.168.67.2

sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 2200 -d 192.168.67.2 -j SNAT --to-source 192.168.67.1
```
I know this is a lot of IP tables rules and it may not make sense initially, but what it's doing is pretty simple: here's a sequential run down

- Line 1: Creating a default rule in the forward  chain to drop packets.
- Lines 2-4: Enabling packet forwarding  coming from eth0 with a destination of 2200 to the wireguard interface 
- Line 5: Change the destination address of packets  destined to port 2200 to the wireguard IP of the home server
- Line 6: Change the source IP so that the home server knows to respond to the VPS.

Try the following command aimed at the public IP of your VPS
```
netcat -vz <VPS_Public_IP> 2200
```
If you are prompted with a success, then you are good to go!

## Extra Credit
IPtables rules are not persistent, so if you would like these rules to survive a reboot install the netfilter-persistent package, save the rules, then enable the service. 
```
sudo apt install netfilter-persistent

sudo netfilter-persistent save

sudo systemctl enable --now netfilter-persistent
```
