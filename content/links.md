---
title: Reference Links
draft: false
---
A page of different reference links that I think are interesting or useful. None of these are sponsored links or anything. This is basically a bookmarks page for when I dont have my bookmarks.

## Security
- Sample Exploitation/Reverse Engineering Practice Binaries: https://crackmes.one/
- OSINT Framework: https://osintframework.com/
- Intel Techniques OSINT Tools: https://inteltechniques.com/tools/index.html
- Digital Forensics and Incident Response Resources: https://www.jaiminton.com/cheatsheet/DFIR/#
- Yubikey SSH Authentication: https://www.yubico.com/blog/github-now-supports-ssh-security-keys/
- Web Application Security and Red Team Resources: https://github.com/swisskyrepo/PayloadsAllTheThings

## Programming
- Project Based Tutorials in C: https://github.com/chesal/Project-Based-Tutorials-in-C
- Python Discord Embed Generator: https://cog-creators.github.io/discord-embed-sandbox/
- Armv7 System Emulator: https://cpulator.01xz.net/?sys=arm-de1soc
- Web Selector Reference: https://www.w3.org/TR/CSS2/selector.html
- Kiwix, Offline repos of resources: https://browse.library.kiwix.org/#lang=eng

## Education
- Database of Free / Open Access Online Computer Science Books, Textbooks, and Lecture Notes: https://www.freetechbooks.com/ 
- Azeria Arm Assembly Tutorials: https://azeria-labs.com/writing-arm-assembly-part-1/
- Odin Project Web Dev Tutorials: https://www.theodinproject.com/paths
- The Book of Secret Knowledge: https://github.com/PR0M37H3U5/the-book-of-secret-knowledge
- Learn X in Y Minutes: https://learnxinyminutes.com/
- The Linux Programming Interface By Michark Kerrisk: https://sciencesoftcode.files.wordpress.com/2018/12/the-linux-programming-interface-michael-kerrisk-1.pdf

## IT
- Cool shop for used hardware: https://labgopher.com/␋
- Server Part Deals: https://serverpartdeals.com/␋
- Speed Tests that arent lying to you: https://speedof.me/␋
- Alt Speedtest: https://testmy.net/␋
- Shucked drive prices:  https://shucks.top/␋
- Headless Raspberrry Pi Setup: https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html␋
- Self Hosted Projects: https://github.com/awesome-selfhosted/awesome-selfhosted␋
- DIY Server Rack Plans: https://tombuildsstuff.blogspot.com/2014/02/diy-server-rack-plans.html?m=1␋
- Online PDF Tools: https://www.ilovepdf.com/␋
- Cheap Dedicated Hosting (Germany/Finland): https://www.hetzner.com/dedicated-rootserver/ ␋
- Cheap Dedicated Hosting (US): https://www.reliablesite.net/dedicated-servers/

## Linux
This is just commands that I either use often or thought were interesting
- Set or remove debugging symbols: `Bash -x/+x`
- Run the Last Command as Root: `sudo !!`
- Start a Web Server on  https://localhost:8080: `python -m SimpleHTTPServer`
- Grab your External IP:  `curl ifconfig.me`
- Query Wikipedia Over DNS: `dig +short txt <keyword>.wp.dg.cx`
- SSH Through a JumpHost:  `ssh -t reachable_host ssh unreachable_host`
- Close shell but preserve child processes: `disown -a && exit`
- Print Slow Typing like the movies: `echo "Im in the mainframe" | pv -qL 10`
- Scroll Random data on the terminal: `cat /dev/urandom | hexdump -C | grep "ca fe`
- Find if your system is 32 or 64 bit: `getconf LONG_BIT`
- Terminal Beep when a machine comes online:  `ping -i 60 -a <IP_ADDR>`
- Create a Random 10 Char String: `strings /dev/urandom | grep -o '[[:alnum:]]' | head -n 10 | tr -d '\n'; echo`
- Poor mans tree:  `find . -type d | sed -e "s/[^-][^\/]*\//  |/g" -e "s/|\([^ ]\)/| - \1/" or  find | sed 's|[^/]*/|- |g'`
- Edit Disks Easier: `cfdisk`

## Etc
- Microsoft Activation Scripts: https://github.com/massgravel/Microsoft-Activation-Scripts
    - For legal reasons, im not promoting the use of this, im just documenting it's existence
