---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Secure Web Application Gateway (SWAG) Setup'
tags:
  - SWAG
  - Reverse Proxy
  - LetsEncrypt
  - Docker
---
## Intro
I was a pretty big user of the LinuxServer.io LetsEncrypt container since it integrated all of the things that I wanted to put in front of my applications into a simple-to-setup container with minimal configuration. Unfortunately, the team has had to transition to a different setup due to a trademark request. Due to my own laziness, I didn't really feel like learning how to use the new system so I ended up doing things the hard way for a couple of years, don't do what I did. If you take the time to learn how to use it, the new container, dubbed SWAG is actually pretty cool. This is how I have it set up in my environment with Cloudflare DNS. 

## What is SWAG?
Swag is the Secure Web Application Gateway. It is the culmination of a couple of different technologies that assist with the secure deployment of web applications to the internet including: Nginx  (web server used for Reverse Proxy services), certbot (automation script used to grab new certificates), fail2ban (web application firewall). I typically like to use the service because I would rather have a SWAG workflow that I can put in front of my web applications rather than have to learn a million individual ways to ensure that SSL is properly set up with my applications. When set up, the only traffic sent via HTTP is local web requests that don't leave the machine itself.

## Setup Instructions
Swag includes the option to deploy via docker-compose which largely simplifies the install process. Here is a sample docker-compose config to get you started.
```
---
version: "2.1"
services:
  swag:
    image: lscr.io/linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN #Required for Fail2ban to modify iptables
    environment:
      - PUID=1000 #Specifying user and group id helps preemptively rectify file permission issues
      - PGID=1000
      - TZ=America/New_York #Sets time zone
      - URL=thaddeuskoenig.com #This should be the root of the domain you own
      - VALIDATION=dns #lets swag know not to use HTTP validation
      - SUBDOMAINS=code-server, #subdomains will be placed in CSV format without spaces with a trailing comma
      - DNSPLUGIN=cloudflare #tells swag to use the cloudflare.ini plugin file
      - EMAIL=admin@thaddeuskoenig.com #Optional Email used for expiration alerts
      - ONLY_SUBDOMAINS=true #optional, alerts swag not to apply  these rules to the root domain
      - STAGING=false #optional, set to true when testing due to LE rate limiting, certs wont validate unless set to false
    volumes:
      - /home/<USER>/docker/swag/config:/config #sets a predictable location for your config files
    ports:
      - 443:443 #redirect 443 from the host to the container
      - 80:80 #optional

    restart: unless-stopped #restart if there is an issue, unless the user manually stops the service
```
After making the appropriate edits to configure the file to your environment save the file as docker-compose.yml. After your file is saved, while you're in t he same directory run the command docker-compose up -d to run the container in the background.

## Post Install Configuration
So after you run the container, you should have a ./config directory sitting in your current working directory.  You will need a few files here for your post-install configuration. Primarily we will need to set up your Cloudflare API validation. Open the file `./config/dns-conf/cloudflare.ini` in your text editor of choice. This file should have a link to instructions on how to set up your token at the time of writing, the link leads [here](https://github.com/certbot/certbot/blob/master/certbot-dns-cloudflare/certbot_dns_cloudflare/__init__.py#L20). It recommends that instead of using the global API token, you enter the CloudFlare Dashboard and configure a unique API token for your DNS Zone with `DNS:Edit` permissions. After gaining the token, you can either remove all the text or comment everything out and include the following line:
```
dns_cloudflare_api_token = <CLOUDFLARE_ZONE_TOKEN>
```
Make sure to save and quit. This will allow for SWAG to use the Cloudflare API to prove DNS ownership. From here you can take advantage of the work that the linuxserver.io team has taken in the way of populating a plethora of popular nginx web application configurations for reverse proxy application. Within the `../config/nginx/proxy-confs` directory there are several sample nginx proxy configurations. They should be ready to go out of the box if your container resides on the same machine and uses the default container name. If not you can open the folder and change the $upstream_app to the IP in which the application can be reached. Once the edits have been made simply rename the file so that it's no longer appended by .sample.  At this point you can restart the container with the following commands and the container should be ready to pull your new certs and redirect HTTPS traffic to your insecure web application. 
```
docker-compose down
docker-compose up-d
```