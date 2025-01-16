---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Testing Drive Read/Write Speed Via Terminal'
tags:
  - DD
  - Drives
  - Linux
  - Legacy
---
Just a quick example of how to use dd to test drive read/write speed. NOTE: Dont do this a lot or you could shorten the live of your drive since youre making arbitrary writes. 

## Reading
```
sudo dd if=/dev/zero of=/tmp/test.img bs=1G count=1 oflag=dsync
```
This command will write 1G of zeros to an arbitrary file of your choosing. Feel free to delete it when you're done. 

Your output will look a little like this.
```
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 1.09503 s, 981 MB/s
```
## Writing
```
sudo dd if=/tmp/test.img of=/dev/zero bs=8k
```
This command will read that same file in 8k chunks similar to how your normal file reads/writes are. 

Your output will look a little like this. 
```
131072+0 records in
131072+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 2.13873 s, 502 MB/s
```
P.S. This is just a test with a bunch of zeros to get a ballpark of your drive speeds. Actual speeds will look different depending on the workload. 
