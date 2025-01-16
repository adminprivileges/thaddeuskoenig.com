---
date: '2023-11-23T00:00:00-05:00'
draft: false
title: 'Clear Home Assistant Refresh Tokens'
tags:
  - Home Assistant
  - Tokens
  - Authentication
  - Legacy
---
As much as I love Home assistant there are some small papercuts that can make it annoying to deal with, this is one of them. Every time you elect to "stay signed in" and you don't sign out before exiting the page you'll have a nice token sitting in this long list of your previous mistakes. I found this quick fix on Reddit, but understand that this will clear ALL the tokens in your home assistant (even the session that youre currently in) so you'll have to log back in. In order to do it, just pop open your web console and paste the following javascript in there, your browser might bark at you saying something about pasting untrusted code, just allow pasting and keep it pushing.

```
hass = document.getElementsByTagName('home-assistant')[0].hass;

hass.callWS({

        type: "auth/refresh_tokens"

    })

    .then(r => r.filter(t => t.type == 'normal').map(t => hass.callWS({

        type: "auth/delete_refresh_token",

        refresh_token_id: t.id

    })));
```