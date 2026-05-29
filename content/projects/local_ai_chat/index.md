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
- URL: example.com
- SUBDOMAINS: ai
- EMAIL: you@example.com

The setup uses `network_mode: host` for ease of use and connectivity. If you are using your machine for more services, you may want to proxy these ports instead. 

### 5. DNS Setup
I use cloudflare for my DNS registrar and from there I can use the API to verfy my domain ownership for the automated certificate generation. Make sure to generate a token that has the permissions to edit the DNS for your zone. 
```
# Start the stack once
sudo docker compose up -d
sudo docker logs -f swag

# SWAG will create its config files under `./swag`
# Edit the Cloudflare DNS Plugin and add your 'dns_cloudflare_api_token' 
vim ./swag/dns-conf/cloudflare.ini
chmod 600 ./swag/dns-conf/cloudflare.ini
sudo docker restart swag
sudo docker logs -f swag
# Wait until you see `Server Started`
```
### 6. Reverse Proxy Config
Create the SWAG Proxy config:`vim ./swag/nginx/proxy-confs/open-webui.subdomain.conf`
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
### 7. DNS
For a DNS setup, set ai.example.com to the IP of your server and connect to it via `https://ai.example.com` .

