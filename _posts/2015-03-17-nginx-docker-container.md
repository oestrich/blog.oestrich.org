---
layout: post
categories:
- docker
- nginx
title: nginx Docker Container
---

As part of my Raspberry Pi cluster (aka "bramble" as I found out recently) I have an nginx load balancer. Here are the steps I took to create it.

##### oestrich/nginx-pi

``` docker
FROM oestrich/arch-pi
MAINTAINER Eric Oestrich "eric@oestrich.org"

RUN pacman -Syu --noconfirm \
  && pacman -S --noconfirm \
    nginx \
  && pacman -Sc --noconfirm

RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log

VOLUME ["/var/cache/nginx"]

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

This starts with the base nginx container, which I took from [here](https://github.com/nginxinc/docker-nginx/blob/master/Dockerfile) and adapted for arch-pi.

##### nginx Dockerfile

``` docker
FROM oestrich/nginx-pi
MAINTAINER Eric Oestrich "eric@oestrich.org"

ADD ssl /etc/nginx/ssl
ADD sites /etc/nginx/sites
ADD nginx.conf /etc/nginx/nginx.conf
```

Next I have a container that will have all of the information required built into it. This is very specific to my cluster and will only be published on the private registry I have.

There are two folders and a file that gets added in. `/etc/nginx/ssl` contains all of the ssl certificates that will be required. `/etc/nginx/sites` has all of the virtual hosts that this nginx container will be serving. Lastly, `/etc/nginx/nginx.conf` is the base nginx configuration.


##### nginx.conf

``` nginx
user root;
worker_processes  2;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;
    gzip  on;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256;
    ssl_session_cache shared:SSL:10m;

    # If you have a lot of virtual hosts, this is required
    server_names_hash_bucket_size  64;

    include /etc/nginx/sites/*;
}
```

This is a pretty simple `nginx.conf` file. I added in my [ssl configuration](https://blog.oestrich.org/2015/01/nginx-ssl-setup/) and that's about it.


##### sites/example.com

``` nginx
upstream website {
  server docker01:5000;
}

server {
  listen 80;
  server_name example.com;
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 443;
  server_name example.com;

  ssl on;
  ssl_certificate /etc/nginx/ssl/example_com.crt;
  ssl_certificate_key /etc/nginx/ssl/example_com.key;

  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://website;
  }

  keepalive_timeout 10;
  server_tokens off;

  add_header Strict-Transport-Security "max-age=31536000";
}
```

This virtual host configuration file sets up a redirect for regular `http` to `https`, and has all server traffic go over `https`. One very important thing to remember when using nginx with proxy passing, make sure you set the `X-Forwarded-Proto` or rails will think you are accessing it via `http` and not `https`. It took a good amount of time to figure that out. [HSTS](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) is also set for a year to make sure clients always return via `https`.

Once all of this is set up you build the container and push it to a private repository. Pull the nginx container to the docker host and install this service file.

##### nginx.service

``` docker
Description=nginx
Requires=docker.service
After=docker.service

[Service]
ExecStart=/usr/bin/docker run --rm -p 443:443 -p 80:80 --name nginx registry.example.com/nginx
Restart=always

[Install]
WantedBy=multi-user.target
```

This service file keeps the nginx container running. It will always restart the service to make sure nginx is running. The `--rm` flag is important so that after the container is stopped it will remove the named container. This allows for the container to start again with the same name when `systemd` starts the service next.

That's it. You should now have an nginx docker container that will serve traffic.
