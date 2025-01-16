---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Sed alternate delimiters'
tags:
  - sed
  - delimiters
  - linux
  - Legacy
---
Sometimes when trying to replace text using the `sed` command line tool you will tun into the issue in which your text has the forward slash characters which can be a bit of a headache because thats what sed uses to delimit strings. Thankfully the delimiter is pretty flexible. I like to use the pipe "|" symbol, but pretty much any other symbol will work such as the tilde "`" or the colon ":" ex:
```
sed -i 's|TEXT|REPLACEMENT_TEXT|g' test.txt

sed -i 's:TEXT:REPLACEMENT_TEXT:g' test.txt

sed -i 's`TEXT`REPLACEMENT_TEXT`g' test.txt
```