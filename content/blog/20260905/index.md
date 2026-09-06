---
date: '2026-09-05T15:50:00-05:00'
draft: false
title: '05SEP2026 - Cleaning Trackers from Links.'
tags:
  - privacy
  - ios
  - link
  - tracking
--- 
# 01. Intro
Usually when I share links to people I try to clean out the tracking infomation. Anyone who has been shared a social media link by somone has likely seen the little indicator of the person who shared them the link. Its just another way that companies track you around the internet to gather information about you. I wont go on too long about my opinions about tracking and ads, I'll just show you the point of the post. See this link here to the live shark cam at the Monterey Bay Aquarium https://www.youtube.com/live/tEtg5Kg3voQ?si=kslcgshej56klows . I have alterred the tracking string, but in this case, the actual usable url is https://www.youtube.com/live/tEtg5Kg3voQ and the tracking info is `?si=kslcgshej56klows`. This tracking string comes in many forms, but it should include a question mark `?` a number of alphabetic characters and an equal sign `=`. Because I get tired of removing this tracking string myself, I wanted a quicker solution. I know apps for this exist, but this is hardly a job for an entire app, this is where IOS shortcuts come in handy. 

# 02. The Shortcut
Here is a link to the shortcut https://www.icloud.com/shortcuts/c4211f0c4e1b41a1908f3dfd0af06c92 . It does the following:  
1. Receive URLs from Sharesheet
    - If there is no input, copy from clipboard
1. Set Veriable `x` to the URL it grabs. 
1. Regex Match for `^(.*?)\?[^=]+=` in the variable `x`. This looks for the preamble of the tracking string in the case of the link from the intro that would be `?si=` and sets the text before it as a group. 
1. Gets the first group (the only group) from the regex match
1. Sets that first group to a variable Final
1. Asks if you want to copy it to the clipboard or go to the URL. 

There isn't much error checking in the script because it was just a quick thing I threw together, but it does work for all the links that I've tried. Hopefully it works for you. 
