---
layout: post
categories:
- nginx
- ssl
title: nginx SSL Setup
---

I reconfigured SSL for nginx this morning and wanted to post the config that got me an "A" on [SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=blog.oestrich.org). I will keep this post updated as I change it over time. Hopefully this will be helpful to someone else who is setting nginx up with SSL.

##### /etc/nginx/config.d/ssl.conf
```nginx
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256;
ssl_session_cache shared:SSL:10m;
```

##### /etc/nginx/sites-enabled/example.com
```nginx
server {
  listen   443;

  ssl on;
  ssl_certificate /path/to/crt/example_com.crt;
  ssl_certificate_key /path/to/key/example_com.key;

  # ...
}
```

For redirecting all traffic on non-SSL to SSL:

##### /etc/nginx/sites-enabled/example.com
```nginx
server {
  listen 80 default deferred;
  server_name example.com;
  rewrite ^(.*) https://$host$1 permanent;
}
```

For creating the crt file, the server's crt should be first followed by a chain to the root CA. I got this flipped a few times and it took a bit to realize they were in the wrong order. It should be noted that `nginx -t` will let you know of this problem.
