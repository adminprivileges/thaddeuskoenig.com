---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Change Proxmox Default Port (Kinda)'
tags:
  - ProxMox
  - nginx
  - Legacy
---
The Proxmox team doesnt really have any plans on changing the default port assigned to Proxmox (8006) and their documentation just tells you to use nginx to proxy the traffic if you want to change the default port so the following script should change your port to the port of your choosing [with 443 as the default]. 
```
#!/bin/bash
#Install nginx
apt install nginx

#Checks for your default nginx file and deletes it
FILE=/etc/nginx/conf.d/default
if [-f "FILE"];then
	rm $FILE
else
	rm /etc/nginx/sites-enabled/default
fi

#Pull the FQDN out of the hosts file
read -p "Enter the FQDN of your server or [ENTER] to set to default from /etc/hosts file" FQDN
if [ -z FQDN ]
then FQDN=$(hostname -f)
fi

#Create your new nginx File
cat > /etc/nginx/conf.d/proxmox.conf << EOF
upstream proxmox {
    server "$FQDN";
}

server {
    listen 80 default_server;
    rewrite ^(.*) https://$host$1 permanent;
}

server {
    listen 443;
    server_name _;
    ssl on;
    ssl_certificate /etc/pve/local/pve-ssl.pem;
    ssl_certificate_key /etc/pve/local/pve-ssl.key;
    proxy_redirect off;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade"; 
        proxy_pass https://localhost:8006;
	proxy_buffering off;
	client_max_body_size 0;
	proxy_connect_timeout  3600s;
        proxy_read_timeout  3600s;
        proxy_send_timeout  3600s;
        send_timeout  3600s;
    }
}
EOF

#test your config
nginx -t
#reload nginx
nginx -s reload

#creating some dependencies via systemd overrides
cat > /etc/systemd/system/nginx.service.d/override.conf << EOF
[Unit]
Requires=pve-cluster.service
After=pve-cluster.service
EOF

systemctl restart nginx
systemctl enable --now nginx
```