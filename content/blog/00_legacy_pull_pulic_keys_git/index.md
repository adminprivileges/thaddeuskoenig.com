---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Pull public keys from Github url'
tags:
  - Github
  - pki
  - keys
  - Legacy
---
If you ever want a simple way to pull your public keys from Github so that you can use your safely stored ssh private keys where you need them. 
```
curl https://github.com/[USERNAME].keys
```
This command will put it in your authorized_keys file which will actually allow you to use them in ssh. 
```
curl https://github.com/[USERNAME].keys | tee -a ~/.ssh/authorized_keys
```

