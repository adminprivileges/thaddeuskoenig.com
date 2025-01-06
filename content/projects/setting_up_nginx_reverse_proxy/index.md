---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Setting Up an NGINX Reverse Proxy'
tags:
  - NGINX
  - Reverse Proxy
  - OpenSSL
  - HTTPS
---
If you've ever set up an application like Plex with a web server on a non-standard port, wanted a bit more security to go into accessing applications over networks that you may not fully trust, or even just wanted to ensure all of your requests to your web services just go to one place so you don't have to mess with firewall rules every time you make something new then this article is for you. My start with NGINX reverse proxies came out of my laziness to remember to specify ports when accessing different apps I had set up and its turned into a powerful tool that allows me to access certain resources that I'm either to lazy to configure for HTTPS or it simply isn't supported. Don't get me wrong, I'm not being disingenuous , if you have an insecure HTTP application, the traffic between the proxy and the app will be via HTTP, but what makes this cool is that the subsequent traffic between you and the proxy is all via HTTPS, meaning as long as you trust your LAN this makes accessing those applications from the greater internet a bit more safe. 
## Install NGINX

This tutorial is gonna be made assuming you're using an Ubuntu, but all of the steps will be the same regardless of operating system except actually installing the package from your distribution's archives.
```
sudo apt install -y nginx
```
Its that simple, I promise  if you want to ensure nginx is installed properly you can run a `nginx -v` or simply browse to the address of the server you installed it on or run a curl on localhost. 

## Configuring NGINX
Now I'm making this assuming you already know about SSL certs and hopefully already have your own (they're free with LetsEncrypt), but if you just want the simplest solution self-signed certs are usually fine if it's just a personal thing. You can generate some self-signed certs using the following command. 
```
openssl req -newkey rsa:4096 \ 

            -x509 \
            -sha256 \
            -days 3650 \
            -nodes \
            -out example.crt \
            -keyout example.key
```
- `newkey rsa:4096` - Creates a new certificate request and 4096 bit RSA key. The default one is 2048 bits.
- `x509` - Creates a X.509 Certificate.
- `sha256` - Use 265-bit SHA (Secure Hash Algorithm).
- `days 3650` - The number of days to certify the certificate for. 3650 is ten years. You can use any positive integer.
- `nodes` - Creates a key without a passphrase.
- `out example.crt` - Specifies the filename to write the newly created certificate to. You can specify any file name.
- `keyout example.key` - Specifies the filename to write the newly created private key to. You can specify any file name.

Next we're going to edit the default NGINX configuration file located at /etc/nginx/sites-enabled/default
- First you're gonna use this directive to redirect any HTTP traffic to HTTPS
```
server {

    listen 80;

    return 301 https://$host$request_uri;

}
```

Next, you're gonna get all the things for SSL set so that your server has the appropriate location for your cert and key as well as settings for negotiation.  
```
...

  listen 443;

  server_name www.example.com;


  ssl_certificate           /etc/nginx/example.crt;

  ssl_certificate_key       /etc/nginx/example.key;


  ssl on;

  ssl_session_cache  builtin:1000  shared:SSL:10m;

  ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;

  ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;

  ssl_prefer_server_ciphers on;

  ...
```
- `server_name www.example.com` is the name of your site. 

- `ssl_certificate /etc/nginx/example.crt` and `ssl_certificate_key /etc/nginx/example.key` are the locations for your certs and keys


Finally, you're gonna want to make sure you get the actual proxy settings set up so NGINX knows how to route your traffic to your end server. 
```
...

location / {


    proxy_set_header        Host $host;

    proxy_set_header        X-Real-IP $remote_addr;

    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_set_header        X-Forwarded-Proto $scheme;

    proxy_pass          http://192.168.1.80:8080;

    proxy_read_timeout  90;


    proxy_redirect      http://192.168.1.80:8080 https://www.example.com;

}

...
```
- `proxy_pass http://192.168.1.80:8080` here you're specifying the actual backend server make sure you don't leave a trailing slash
- `proxy_redirect http://192.168.1.80:8080 https://www.example.com` here this is where you're actually telling NGINX to port traffic from http://192.168.1.80:8080 to https://www.example.com