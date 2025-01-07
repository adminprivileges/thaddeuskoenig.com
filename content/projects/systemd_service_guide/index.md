---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Systemd Service Guide'
tags:
  - Linux
  - Systemd
  - Services
  - Python
---
When creating applications that you'd like to run over a continuous period of time, you're eventually going to need to worry about timing, system integration, and startup/exit behavior, but while creating your own methods to do so I'm sure you'll learn a lot about the Linux operating system, but often times its a bit more work than one would like just to get a simple application running. For this tutorial I'm going to show you how I created a service to run my [Eva Unit-01 discord bot](https://github.com/adminprivileges/Eva-Unit-01). Eva is a python script that runs out of a virtual environment. I would like for it to be run on startup with any issues or output logged to the systemd journal. 

Creating the Service
So while "creating a service" sounds like a high task, its as simple as creating a text file in the `/etc/systemd/system` directory. So were going to `sudo vim /etc/systemd/system/evabot.service` My service file is going to look like this. 
```
[Unit]
Description=Thaddeus's Eva Bot
After=network.target

[Service]
User=thaddeus
Type=simple
WorkingDirectory=/home/thaddeus/EVA
ExecStart=/home/thaddeus/EVA/bin/python3 /home/thaddeus/EVA/eva.py
Restart=always
StartLimitAction=reboot

[Install]
WantedBy=multi-user.target
```

- `[Unit]` is simply gonna be comprised of attributes like your services description, dependencies, and behavior. 
  - **Description** is simply a simple description of your service that will show up when you systemctl status evabot
  - **After** tells the system to only start evabot after the network service as started 
 - **Requires** tells the system dependencies that the service requires to run, in my case evabot doesn't have any. 
  - **Wants** is where we will list "soft" dependencies, or things that the service requires to run properly, but wont outright fail without them
- [Service] is where youre going to actually specify the commands that will be run by this service in my case i need to run my python file in the virtual environment. 
  - **ExecStart** - Is the command to be run upon service startup 
  - **ExecStartPre** and ExecStartPost can be used to set behavior pre and post start respectively. 
  - **ExecStop** - The command to be run on service stoppage
  - **ExecRestart** - The command to be run upon a service restart.  
- **User** is simply the user that you would like the service to run under, my service can easily be run my my non-privileged user thaddeus try not to run things as root unless you really need to. 
- **Type** alerts the system of the expected behavior of your service my categorizing it into a behavior type.
  - **simple** - The default behavior, when this is set, the command in ExecStart will be considered the main service.
  - **forking** - When this is set, the ExecStart command will be expected to fork and prodice a child which will in turn be treated as the main process
  - **oneshot** - This is the default is both Type and ExecStart arent set, the process is simply set to do it job once and exit unless the RemainAfterExit flag is set to yes. 
  - **dbus** - This option runs a lot like simple with the exception that the daemon is expected to get an name specified in the BusName option from Dbus subsequent units are  only launched after the name is acquired. 
  - **notify** - similar to simple, with the exception that the daemon is supposed to send a notification via sd_notify before launching subsequent units. 
- `Restart` - by default, if your service fails, systemd will not attempt to restart your process unless told to do so by this flag, the always option tells systemd to restart regardless of exit status, but on-failure only restarts if the exit status is zero
  - **RestartSec** -Specifies the amount of time to wait before restart, the default behavior is 100ms if not specified. 
- **StartLimitAction** - specifies behaviors you system can do if systemd exhausts its retries, in my example the reboot option tells systemd to restart the server. 
  - **StartLimitIntervalSec** - by default if your Restart option is set, systemd will stop trying after 5 failures within a 10 sec interval, if you'd like it to keep going, set this to zero
- `[Install]` is going to be used to set service installation options 
  - **Alias** - can be used to set a nickname for your service that can be recognized by systemctl (for some reason it doesnt work with enable)
  - **Requiredby/WantedBy** - similar to the required and wanted in the Unit section in this case with multi-user .target im saying to only start at runlevel 2/3/4 (a non-graphical multi-user shell) which is where your typical server is gonna run.

## Starting and enabling the service
Now that you've created your text file in the appropriate directory, youre gonna want to reaload the systemd manager config using 
```
sudo systemd daemon-reload  
```
And youre going to start and enable your service using
```
sudo systemctl enable evabot && sudo systemctl start evabot
```
of you would like to check the status of your service you can always 
```
sudo systemctl status evabot
```
And its that easy, you did it. 