---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Automating pfSense Key Export'
tags:
  - pfSense
  - ssh
  - pki
  - LetsEncrypt
---
So having keys via LetsEncrypt is pretty cool given the fact that they're free, but unlike other certs these are only good for about 90 days, and while manually moving keys over a couple times a year is not the most annoying thing in the world, I'm pretty lazy so i like to automate trivial tasks as much as possible. I'm doing this with the assumption that you've already set up your certs using the pfSense ACME client so what we're going to be doing is as simple as using rsync to ensure that these files are always up to date. 

## Setting up SSH
So the first thing you're going to want to do is set up ssh keys with whatever the client machine that you would like to send these SSL certs to, this can be done by 
```
ssh-keygen

ssh-copy-id <USERNAME>@<IP>
```

If you have any issues with your ssh-agent [this might help](https://www.thaddeuskoenig.com/home/random-tipsguides/ssh-auth-agent-issues-on-freebsd). 

## Finding your Let's Encrypt Certs
Next you're going to want to find your .key and .crt files, typically they're going to be stored in the /cf/conf/acme/ directory with the name of the domain you're serving. if the files aren't there you can always use the find command to locate your certs `find / -name <DOMAIN>.crt.`

## Using rsync to sync the files
So originally I did this with scp until i saw that doing this with rsync was significantly cleaner so with a simple command line script you can make sure these are always up to date using: 
```
rsync -av  /cf/conf/acme/<DOMAIN>.crt <USERNAME>@<IP>:<TARGET_DIRECTORY>.crt

rsync -av  /cf/conf/acme/<DOMAIN>.key <USERNAME>@<IP>:<TARGET_DIRECTORY>.key
```

## Using cron to automate rsync
Install cron on your pfSense web app by going to System>Package Manager>Available Packages, then access the service using Services > Cron > Settings and add a cron job syncing your .crt and .key files to your target machine at your specified interval, I choose to do mine daily, which may be a bit overkill, but you can chose whats most comfortable to you. in cron you typically want to use full paths so your job is going to look like this 

```
0 7 * * * 	root /usr/local/bin/rsync -az /cf/conf/acme/<DOMAIN>.crt <USERNAME>@<USERNAME>@<IP>:<TARGET_DIRECTORY>.crt

0 7 * * * 	root /usr/local/bin/rsync -az /cf/conf/acme/<DOMAIN>.key <USERNAME>@<USERNAME>@<IP>:<TARGET_DIRECTORY>.key
```

## Note
If you don't want your pfSense box to have a bunch of SSH keys to devices in your network a potentially more secure solution could be simply doing this to send these files to a web server and just hosting it there and simply have the clients go a curl or a wget to pull the files. 