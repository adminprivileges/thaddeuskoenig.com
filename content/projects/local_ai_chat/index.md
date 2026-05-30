---
title: "A complete guide to setting up a local AI Chatbot"
date: '2026-05-29T12:00:00-05:00'
draft: false
tags:
- local-ai
- ollama
- open-webui
- docker
- swag
- searxng
- homelab
---
## Overview
For school and personal curiosity I wanted to mess around with hosting my own local AI Chat server for testing and evaluation. I wanted this to be open source, private, and privacy concious. This setup enables a LAN-locked AU chat web UI backed by Ollama models with the ability to enable web search, code execution, and HTTPS through a reverse proxy. Its important to note, that this is simply meant to be something contained and not sitting on the public internet.

The stack uses Ollama as the local model, Open WebUI as the browser chat interfacem SearXNG as the local metasearch backend, Open Terminal for the code execution environment, and SWAG as an HTTPS reverse proxy. It is important to note that Ollama is run natively on my Ubuntu 24.04.04 LTS host via systemd to reduce issues accessing the GPU. All other components are run in docker. The SWAG reverse proxy is not a necessity, but I belive its best practice with the nature of the application handling logins, API tokens, and executing (somewhat) arbitrary code. With this setup, the only exposed service should be SWAG  
 
In production, the networking looks a bit like this
```
Client Browser -> SWAG Reverse Proxy (htsps://ai.example.com) -> Open WebUI(http://localhost:8080) -> Ollama (http://localhost:11434)
# Optional Extras
- SearXNG (https://localhost:8081)
- Open Terminal (https://localhost:8000) 
```

Since I just repurposed my old gaming PC, my local resources are as follows
- OS: Ubuntu 24.04.04 LTS
- GPU: NVIDIA 1080TI (11GB)
- CPU: Intel i7-7700K
- RAM: 24GB (DDR4) 

## Privacy and Security Considerations
At its core this is an private offline AI Chat tool, the more functionality you add, the more attack surface and privacy you give up. If the prompt is answered entirely by Ollama, then nothing leaves the local machne. If you enable web search, with this setup, your search terms are sent to SearXNG, which will send it to uipstream search engines meaning any prompts that use web search arent entirely local. If Open Terminal is enabled, the model can ask to execute code in a container. this inherently changes the risk profile. While the code is containerized, you should understand the implications of running arbitrary code so be careful with the prompts you create and never enable this for a user you dont completely trust. 

When the stack is initialized, the first user to login is the administrator account, so make sure to immediately login to set your credentials and immediately disable open registration via `Admin Panel` > `Settings` > `General` > `Enable New Sign Ups`. Lastly, since I have my own DNS resolver my DNS address `ai.example.com` simply resides on my local DNS resolver and is not made available to any public records. Don't put this address on your public resolver unless you plan on making this publicly available and understand the implications of doing so. If you plan on doing this I recommend using something like Cloudflare Access. 

## Install

### 1. Install Docker on Ubuntu
The docker version in the Ubunutu repo can get a bit stale, especially if youre like me and ride LTS distros until they arent supported. Its a good idea to just add the Docker Repo directly and get the lastest packages from there. I will refrain from posting the install instructions for that, because it can change. You can visit the docker documentation at https://docs.docker.com/engine/install/ubuntu/ .  
If you are lazy, you can use the official script.  
```
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```

### 2. Install Ollama
For the same reasons I will simply give you the current install instructions for Ollama here, which will walk you through the install and systemd setup https://docs.ollama.com/linux .  
Here is the script for Ollama as well
```
curl -fsSL https://ollama.com/install.sh | sh
```
Make sure the service is started and enabled
```
sudo systemctl enable --now ollama 
sudo systemctl status ollama --no-pager
```
Then you can test the local API to ensure it works
```
curl http://127.0.0.1:11434/api/tags
```
From here, you can pull a model to make sure you can 
```
ollama pull qwen3:8b
```
Because I am using an older system, I am using slightly older models to prevent the system from falling over. Here are some other models I pulled for some variety and testing.
```
ollama pull llama3.1:8b 
ollama pull qwen2.5-coder:7b 
ollama pull phi4-mini
```
If you are like me and have a more modest system, here are some other settings you can enable. Run `sudo systemctl edit ollama` and add the following to keep the models from eating each other. 
```
[Service]
Environment="OLLAMA_KEEP_ALIVE=0"
Environment="OLLAMA_NUM_PARALLEL=1"
Environment="OLLAMA_MAX_LOADED_MODELS=1"
Environment="OLLAMA_MAX_QUEUE=4"
```

### 3. Create Project Directory
From here I created a directory so that all my necessary files are in the correct location. 
```
# Create the directory for the stack
mkdir -p ~/docker/openwebui
cd cd ~/docker/openwebui

# Create the environment file with your secrets
printf "WEBUI_SECRET_KEY=%s\n" "$(openssl rand -hex 32)" > .env
printf "OPEN_TERMINAL_API_KEY=%s\n" "$(openssl rand -hex 32)" >> .env
# Your WEBUI_SECRET_KEY is persistent, so dont regen this key after the initial time

# Create SearXNG config
mkdir -p ~/docker/openwebui/searxng

cat > searxng/settings.yml <<EOF
use_default_settings: true

server:
  bind_address: "0.0.0.0"
  secret_key: "$(openssl rand -hex 32)"
  limiter: false
  public_instance: false

search:
  safe_search: 1
  formats:
    - html
    - json
EOF
```

### 4. Setup the Docker Compose 
Create `docker-compose.yml`:

```
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    network_mode: host
    env_file: .env
    environment:
      OLLAMA_BASE_URL: http://127.0.0.1:11434
      WEBUI_URL: https://ai.example.com

      ENABLE_WEB_SEARCH: "True"
      WEB_SEARCH_ENGINE: "searxng"
      WEB_SEARCH_RESULT_COUNT: "3"
      WEB_SEARCH_CONCURRENT_REQUESTS: "2"
      WEB_LOADER_CONCURRENT_REQUESTS: "3"
      SEARXNG_QUERY_URL: "http://127.0.0.1:8081/search?q=<query>"

      ENABLE_CODE_INTERPRETER: "True"
      CODE_INTERPRETER_ENGINE: "pyodide"
      ENABLE_CODE_EXECUTION: "True"
      CODE_EXECUTION_ENGINE: "pyodide"
    volumes:
      - open-webui:/app/backend/data
    restart: unless-stopped
    depends_on:
      - searxng
      - open-terminal

  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    ports:
      - "127.0.0.1:8081:8080"
    volumes:
      - ./searxng:/etc/searxng:rw
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"

  open-terminal:
    image: ghcr.io/open-webui/open-terminal
    container_name: open-terminal
    ports:
      - "127.0.0.1:8000:8000"
    environment:
      OPEN_TERMINAL_API_KEY: ${OPEN_TERMINAL_API_KEY}
    volumes:
      - open-terminal:/home/user
    restart: unless-stopped

  swag:
    image: lscr.io/linuxserver/swag:latest
    container_name: swag
    network_mode: host
    cap_add:
      - NET_ADMIN
    environment:
      PUID: 1000
      PGID: 1000
      TZ: America/New_York

      URL: example.com
      SUBDOMAINS: ai
      VALIDATION: dns
      DNSPLUGIN: cloudflare
      EMAIL: you@example.com
      ONLY_SUBDOMAINS: "true"
      STAGING: "false"
    volumes:
      - ./swag:/config
    restart: unless-stopped

volumes:
  open-webui:
  open-terminal:
```

Ensure that you change the following values
- WEBUI_URL: https://ai.example.com 
  - Change this to the URL that you plan on using for your site
- URL: example.com
  - Change this to the domain you will be using for cloudflare DNS
- SUBDOMAINS: ai
  - Change to the subdomain you want to use for the site
- EMAIL: you@example.com
  - Change this to the email you use for cloudflare

The setup uses `network_mode: host` for ease of use and connectivity. If you are using your machine for more services, you may want to proxy these ports instead. 

### 5. DNS Setup
I use cloudflare for my DNS registrar and from there I can use the API to verfy my domain ownership for the automated certificate generation. Make sure to generate a token that has the permissions to edit the DNS for your zone. Cloudflare really likes to change the process for doing stuff in the web UI so I will link the docs to the process here: https://developers.cloudflare.com/fundamentals/api/get-started/create-token/ . 
```
# Start the stack once
sudo docker compose up -d
sudo docker logs -f swag
```
SWAG will create its config files under `./swag`, Edit the Cloudflare DNS Plugin and add your `dns_cloudflare_api_token`
```
vim ./swag/dns-conf/cloudflare.ini
```
From here you should be able to restart SWAG without affectin the rest of the stack.
```
# Change the permissions on the file so you dont get yelled at
chmod 600 ./swag/dns-conf/cloudflare.ini
# Retart swag and check your logs
sudo docker restart swag
sudo docker logs -f swag
```
Once the logs say **Server Ready**, you should be good to go. 

### 6. Reverse Proxy Config
Create the SWAG Proxy config: `vim ./swag/nginx/proxy-confs/open-webui.subdomain.conf`
```
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name ai.*;

    include /config/nginx/ssl.conf;

    client_max_body_size 0;

    location / {
        include /config/nginx/proxy.conf;

        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```
Restart SWAG:
```
docker restart swag
```
Check that nginx is listening:
```
sudo ss -tulpn | grep -E ':443|:80'
```
Check logs:
```
docker logs --tail=100 swag
```
If you see any duplicate errors in the logs, its because the default SWAG `proxy.conf` likely has the offending line, so you can just remove it from your config and be good to go. 

### 7. DNS
For a DNS setup, set ai.example.com to the IP of your server and connect to it via `https://ai.example.com` . Make sure to run `nslookup ai.example.com` to ensure this DNS record is working and coming from the correct address. If this command resolves, but you still arent able to access the site, your web browser may be bypassing your local DNS records in favor of its DNS over HTTPS provider.

### 8. First login / Initial Setup
1. Open `https://ai.example.com`
1. Create your initial account **This will be your admin account**.
1. Go to `Admin Panel` -> `Settings` -> `General`
    - Make sure your webui URL is set to `https://ai.example.org`
1. Go to `Admin Panel` -> `Settings` -> `Web Search`
    - Ensure the following are set
      ```
      Enable Web Search: On
      Search Engine: searxng
      SearXNG Query URL: http://127.0.0.1:8081/search?q=<query>
      Result Count: 3
      ```
1. Go to `Admin Panel` -> `Settings` -> `Integrations` -> `Open Terminal`
    - Ensure the following are set
      ```
      URL: http://127.0.0.1:8000
      Auth Type: Bearer
      API Key: value of OPEN_TERMINAL_API_KEY from .env
      ```
        - If you need the key, run the command `grep OPEN_TERMINAL_API_KEY ~/docker/openwebui/.env`
    - Enable the configured terminal


## Testing
At this point you are done. Here are some commands you can use for testing the individual components to ensure they are working as intended. 
- Test Ollama
  ```
  curl http://127.0.0.1:11434/api/tags
  ```
- Test Open WebUI
  ```
  curl -I http://127.0.0.1:8080
  ```
- Test SearXNG:
  ```
  curl -sG "http://127.0.0.1:8081/search" \
    --data-urlencode "q=open webui" \
    -d "format=json" | head
  ```
- Test Open Terminal:
  ```
  curl -s http://127.0.0.1:8000/health
  ```
- Test HTTPS (From a remote computer):
  ```
  curl -vk https://ai.example.com
  ```
- Check active listeners:
  ```
  sudo ss -tulpn | grep -E ':443|:80|:8080|:8081|:8000|:11434'
  ```
- Check containers:
  ```
  docker ps
  ```
- Check Open WebUI logs:
  ```
  docker logs --tail=100 open-webui
  ```
- Check SWAG logs:
  ```
  docker logs --tail=100 swag
  ```
### Troubleshooting
while the process is pretty automated, there can still be issues. Here are some things I ran into during setup, or issues I could forsee. 

#### Firefox says it cant connect to server
This usually means firefox isnt able to open a TCP connection at all so the server isnt even responding (or it cant reach it). 
- Check DNS from the remote client.
  ```
  nslookup ai.example.com
  ```
  - If this does not resolve to your server's IP, you need to fix your local DNS resolution.
  - If it does work run the command `curl https://ai.example.com` the results will help you find the problem
    - Could not resolve host (DNS Problem)
    - Connection refused (Able to reach machine, but nothing is listening on HTTPS)
      - Check the SWAG logs to ensure that nothign stopped it from starting
    - Connection timed out (firewall, routing issues, service unreachable)
      - double check your networking settings to ensure you are able to reach this machine on port 443 and you arent being blocked. 
    - SSL Certificate error(SWAG issue)
      - open your `./swag/nginx/proxy-confs/open-webui.subdomain.conf` and ensure that everything is spelled right and you have the correct key in `./swag/dns-conf/cloudflare.ini` that has permissions to edit DNS settings for your zone. 
    - 502 Bad Gateway
      - SWAG is working, but is unable to reach Open WebUI. Check the docker logs for the Open WebUI container to ensure it started correctly.

#### SWAG not listening
Double check the docker logs with `sudo docker logs --tail=100 swag`. If the nginx is invalid, SWAG will start, but it wont serve the site. I hit this error a couple times `"proxy_http_version" directive is duplicate`. This is because out of habit I was manually including directives I used in the past, but SWAG is pretty smart and its already including them with `include /config/nginx/proxy.conf;`. The fix was to use the more minimal config above. If SWAG updates in the future to include more directives in proxy.conf, just remove the offending lines and restart swag after the changes with `sudo docker restart swag`.

#### SWAG showing 502 Bad gateway
This means that the browser reached SWAG, but SWAG could not reach Open WebUI. 

Check Open WebUI:
```
sudo docker ps
curl -I http://localhost:8080
```
If Open WebUI is not responding, check the logs for any errors:
```
sudo docker logs --tail=100 open-webui
```
If everything looks okay, make sure to double check the proxy target in your `./swag/nginx/proxy-confs/open-webui.subdomain.conf` it should be:
```
proxy_pass http://127.0.0.1:8080;
```

## Maintenance
Back up Open WebUI data before major updates:
```
docker run --rm \
  -v open-webui:/data \
  -v "$PWD":/backup \
  alpine \
  tar czf /backup/open-webui-backup.tar.gz -C /data .
```
Back up SWAG config:
```
tar czf swag-config-backup.tar.gz swag
```
Check disk usage occasionally:
```
docker system df
du -sh ~/.ollama
```
Remove unused Ollama models:
```
ollama list
ollama rm model-name
```
Updating Containers
```
docker compose pull
docker compose up -d
docker image prune
```
