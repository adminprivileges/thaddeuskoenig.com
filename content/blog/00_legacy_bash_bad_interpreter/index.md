---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: '/bin/bash^M: bad interpreter'
tags:
  - bash
  - linux
  - Legacy
---
Bash scripts are very sensitive to line endings which can cause some portability issues between windows and unix-like systems (depending on how the text editor encodes line breaks). If you would like to see the invisible characters that are making your life confusing simply type: 
```
cat -v <FILE>
```
The easiest solution to this issue is a simple sed replace line:
```
sed -i -e 's/\r$//' <FILE>
```