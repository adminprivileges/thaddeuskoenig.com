---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: '400: Bad Request in HomeAssistant'
tags:
  - HomeAssistant
  - Web Apps
  - NGINX
  - Legacy
---
So for some reason in v2021.7.0 HomeAssistant introduced a bug that breaks a lot of systems that rely on its NGINX reverse proxy add-on to provide ssl capabilities. Thankfully the fix itself is pretty simple. 

- To begin, try to navigate to the site to produce the error, then in the HAS web GUI navigate to the Supervisor Logs (**Settings**>**System**>**Logs**)

- Grab the IP that shows up in the error that reads 

  ``` 
  "A request from a reverse proxy was received from XXX.XXX.XXX.XXX, but your HTTP integration is not set-up for reverse proxies" 
  ```

- From here navigate to your configuration file in your chosen method (/config/configuration.yaml) where we will make edits.

- Add the following line, including the IP you grabbed from the logs
```
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - XXX.XXX.XXX.XXX 
```
- `Reboot` the machine