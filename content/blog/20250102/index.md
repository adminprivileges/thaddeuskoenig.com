---
date: '2025-01-02T16:04:07-05:00'
draft: false
title: '02JAN25 - NixOS'
tags:
  - NixOS
  - Declaritive Build
  - Immutable Operating Systems
  - Flakes
---

So I didn't do anything today so I'm going to take today to write about NixOS. I started using NixOS recently because I would like to incorporate declarative and immutable operating systems into my research. I was introduced to NixOS via one of my favorite podcasts [Linux Unplugged](https://linuxunplugged.com/). I wasn't initially sold on it because it sounded like a fad, but so far its pretty cool. This is how I understand it. 


Theres a difference between Nix and NisOS. Nix is a package manager. At the time of writing it has eclipsed the AUR as the largest Linux repo. You can use Nix and not have to use NixOS, ive heard people using it on other Linux Operating systems and even on Mac. I need to figure out how the packaging works exactly so that I can understand why this works, because I think that's pretty powerful that it does. NixOS is the entire operating system. From what I understand NixOS is atomic, meaning all changes to the operating system happen at once so updates work in a copy-on-write fashion where updates are applied as an entire system snapshot. The system itself is read only to the actual user and any actual system changes are made via the `/etc/nixos/configuration.nix` file. Any time you make changes to  your system it is built from scratch and put in a read only state. Files in your home directory are safe from this destructive change meaning you can roll back to a previous iteration of you OS but still keep your files. 


Now lets talk about Flakes, I am by no means an expert on this, nor am I sure that I even understand it fully quite yet but flakes can allow an additional layer of reproducibility to your NixOS. Because you dont specify package versions when you download packages, if I wanted to work on a specific config such a jellyfin server sitting behind an nginx proxy, I could create a configuration file, in this case a `flake.nix` to make sure these two things are installed and configured correctly. This setup has no guarantees to ensure that the versions are the same when someone else uses my config file as they will simply be downloading the lastest versions of the applications. Flakes step in to help that by using a system of inputs and outputs which  define the repository locations and the development environments respectively (im sure it does other stuff too). It then goes about creating a `flake.lock` file from your configuration, which will be a listing of the exact versions of the applications installed ensuring that when someone else installs it they will get the exact same setup and not just the latest version.   

To see nix in action check out [my NixOS config](https://github.com/adminprivileges/nixos) here. It is a pretty basic setup so far, but I hope to add more in the future as I understand it more. 
