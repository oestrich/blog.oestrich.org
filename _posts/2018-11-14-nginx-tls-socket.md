---
layout: post
categories:
- nginx
- tls
title: Nginx TLS Socket Termination
description: How to set up nginx to be an TLS socket for standard TCP sockets
date: 2018-11-14 9:00 PM EDT
---

For [MidMUD][midmud], I have a secure endpoint that you can connect to the game with. This is simply a TLS wrapper around a standard telnet connection. It's very simple to set up, and assumes that you already have Let's Encrypt set up on your main domain.

You can also load balance with this, which MidMUD uses to balance across the cluster.

This must be set up at the top level `nginx.conf` as there are no virtual host semantics for raw sockets.

#### nginx.conf

```nginx
# other config

stream {
  upstream telnet {
    server game-01.example.com:5555;
    server game-02.example.com:5555;
    # add as many or as few of these as you need
  }

  server {
    listen [::]:5555;
    listen 5555;

    proxy_pass telnet;
  }

  server {
    listen [::]:5443 ssl;
    listen 5443 ssl;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    proxy_pass telnet;
  }
}
```

This has nginx listening on `5555` and `5443` as a plain text TCP socket and a secure TLS socket. Both of which proxy to the local TCP socket which should be in your secure network, and may also be on the same host.

Hopefully this helps you configure your own plaintext TCP socket to be more secure!

[midmud]: https://midmud.com/
