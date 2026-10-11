---
date: '2026-10-11T07:31:00-05:00'
draft: false
title: '11OCT2026 - Downloading All of Wikipedia.'
tags:
  - wikipedia
  - offline
  - prepping
--- 
# 01. Intro
This isnt something thats immediately practical, but I thought it was interesting party trick. I saw this on the Proton youtube page here: https://youtu.be/3VJ5Htyhm7k?si=LTC5sgaAbNU-b6Ff . The video itself is a bit gimmicky, but I still enjoyed it so I  decided to check out how to do the wikipedia thing. Its actually pretty simple, you just download a special browser and a (pretty big) file to search using the browser. The browser id developed by a group called Kiwix, which has a mission to bring tech to places that don't have it and making free education offline and accessible, check them out here and you can donate if you are so inclined: https://get.kiwix.org/en/about-us/ .   
# 02. Walkthrough
This wont really have muuch going on here, because it really is just downloading two things and getting going.
1. Download the Kiwix reader from https://get.kiwix.org/en/solutions/applications/kiwix-reader/ . If you are running linux, then it is also packaged in many popular repos, for example, the ubuntu installation is:
    ```
    sudo apt install kiwix
    ```
1. Once you have the browser installed, you can navigate to the kiwix catalogue here to find content https://browse.library.kiwix.org/#lang=eng . The wikipedia download is currently about 120GB, but they have smaller versions available to save space by doing things like removing images or only including popular pages. The current direct download is here: https://lb.download.kiwix.org/zim/wikipedia/wikipedia_en_all_maxi_2026-08.zim .
1. Once you have it all installed, just open the kiwix browser and navigate to your .zim file and boom, you have a complete offline wikipedia download for the apocalypse.
