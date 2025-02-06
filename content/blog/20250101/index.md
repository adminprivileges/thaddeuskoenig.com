---
date: '2025-01-01T10:04:07-05:00'
draft: false
title: '01JAN25 - Hugo Site Plans'
tags:
  - CI/CD
  - GitHub
  - GitHub Pages
  - GitHub Actions
---

Hello, this is the beginning of my (somewhat) daily blog. The intention of this blog is less of an exercise in writing about my life and more of a exercise in making a habit of documentation. I plan to write daily about lessons I learned about computers that day and I'm sure there will be a non-computer related blog or two. When I implement tagging and search, I will make sure that things are tagged appropriately. Now let's get into the first blog post:

 I would like to update my site from my current Google Sites posting to a site dynamically built via Hugo, delivered through a [GitHub CI/CD pipeline](https://github.com/resources/articles/devops/ci-cd) (maybe a [Gitlab](https://about.gitlab.com/company/) one in the future), and ultimately served over either [GitHub Pages](https://pages.github.com/) or [Cloudflare Pages](https://pages.cloudflare.com/). I settled on Hugo after I heard about it on one of my favorite podcasts (Linux Unplugged)[https://linuxunplugged.com/] since they have a similar setup for their site (JupiterBroadcasting.com)[https://github.com/JupiterBroadcasting/jupiterbroadcasting.com] I plan to take it further, mainly as an academic exercise, but also to have a method of deploying my site that allows me a bit more flexibility and a lot more pizzaz. Once I learn the GitHub side of things, I plan to tear it down and rebuild it on [Gitlab Pages](https://docs.gitlab.com/ee/user/project/pages/) for a fully open-source tech stack. 

So far I installed Hugo, but I kept running into issues working through the QuickStart for the [theme](https://github.com/schnerring/hugo-theme-gruvbox) I was trying to use which led to some frustration. From what I understand there's a couple different ways for me to run Hugo on my system

  1. **Install it from the package repo**

  - This is my usual go to. I was attempting this on my Ubuntu 24.04 laptop but I ran into a couple issues. Hugo is written in (Go)[https://go.dev/] and as a consequence of that, the things you want to do in are dependent on the version of Go you have on your system The package maintainers are pretty good at ensuring the version of Hugo in the package repo does not rely on a Go version that isn't in the package repo. Theme developers are not the same (and nor should they have to be), and that's the issue that I face today. In order to install and run the theme so that I can edit and develop locally, I need a version of Go that is more updated than the one in the repos. I usually don't have this issue much, but choosing an LTS has really bit me. 

  2. **Install the Snap**

  - Usually if something like this happens, I check the [Snap repository](https://snapcraft.io/) because projects are sometimes nice enough to make a snap available that has all the necessary components bundled up. A lot of people don't like snaps, but I'm pretty happy using them when they're available. I usually keep systems installed for long periods of time and will upgrade instead of doing a nuke and pave when the time has come for me to do so. Snaps have helped me reduce overall system crud to aid in this endeavor. Unfortunately this did not work either since the Hugo snap is bundled with Go and that version is older too, installing the Go snap (which is the correct version) doesn't really help with that either because the snap isn't looking there.    

  3. **Build From Source**

  - I don't like building from source, because cleaning up after myself sucks, also I want to upgrade my packages with the system. Maybe I just need to learn the process better to get more comfortable with it

These were my issues with Hugo. My solution was to use [NixOs](https://nixos.org/) I have NixOS installed on an old laptop because I'm trying to learn more about it to integrate immutable and declarative operating systems into my current research for school and I have got to say that its pretty incredible. Getting this project started on NixOS was as simple as adding a couple lines to my config to install Hugo, Go, and [NodeJS](https://nodejs.org/en/about) and running through the QuickStart guide. While I understand that the fix was not unique to NixOs, and that I would have had similar success with a rolling distro like [Arch Linux](https://archlinux.org/about/) I had NixOs already around and it worked well, I'm becoming more and more sold on the idea of NixOS so I'm happy to be finding more uses for it in my daily life. 
