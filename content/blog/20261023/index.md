---
date: '2026-10-23T18:37:00-05:00'
draft: false
title: '23OCT2026 - Google Home Music Assistant Setup.'
tags:
  - smart home
  - google home
  - HomeAssistant
  - MusicAssistant
--- 
# 01. Intro
I got some google home max speakers a while back because the hardware on them is very good, but they've gotten a lot worse since they released Gemini. I'm opted out of Gemini, because I dont really want to use their AI stuff, and ive just been very frustrated with the product in full. Ive had them firewalled off from the internet and just use them as bluetooth speakers. I found out about MusicAssistant, so now I want to use them as a Sonos like experience. I have them connected to Raspberry Pis now with 2.5mm aux chords and I will use them to send music to the speakers.

# 02. Steps
1. Install Squeezelite. This is the app that MusicAssistant will subscribe to.
```
thaddeus@pi01$ sudo apt install squeezelite
```
1. Grab your device ID from aplay.
    ```
    thaddeus@pi01$ aplay -l
    **** List of PLAYBACK Hardware Devices ****
    card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
      Subdevices: 7/8
      Subdevice #0: subdevice #0
      Subdevice #1: subdevice #1
      Subdevice #2: subdevice #2
      Subdevice #3: subdevice #3
      Subdevice #4: subdevice #4
      Subdevice #5: subdevice #5
      Subdevice #6: subdevice #6
      Subdevice #7: subdevice #7
    card 1: vc4hdmi [vc4-hdmi], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    ```
    - *In this case, the hardware device I want is the audio jack at Card 0 Device 0 (hw 0,0)*
1. Test that your device is working like you want it by running squeezelite manually in debug (`-s <IP>` specifies the musicassistant IP). The error reporting in the systemd service isnt very verbose. If you see "Connected", youre good to go. If you see `output_init_common:401 unable to open output device` then double check your device. 
```
thaddeus@pi01:~ $ sudo squeezelite -o hw:0,0 -s 192.168.1.100 -d slimproto=sdebug                                                                                                   
[19:04:10.949958] slimproto:903 connecting to 192.168.1.100:3483                                                                                                                   
[19:04:10.967339] slimproto:942 connected          
```
1. Edit `/etc/default/squeezelite` to specify your speaker and give it a name and set the hardware ID. (SB_SERVER_IP is optional, but i dont like autodiscovery).
```
SL_NAME="Raspberry Pi Speaker"
SL_SOUNDCARD="hw:0,0"
SB_SERVER_IP="192.168.1.100"
```
1. Restart Squeezelite. 
```
sudo systemctl restart squeezelite
```
1. The Raspberry Pi has an internal volume control that i set at 90% so that I can ensure music assistant controls the full volume (I do 90% because I dont want to accidentally blow out my speaker).
```
# Run this command
amixer -c0 set 'PCM' 90%
# Edit Crontab 
crontab -e
# insert this into the crontab file
@reboot sleep 10 && amixer -c0 set 'PCM' 90%
```
# 03. Follow-up
From here, your device should show up in Music Assistant. From there, you can play music from it. I really like the Music Assistant framework and I might look into adding mics to the PIs to make them local voice assiatants in the future.
