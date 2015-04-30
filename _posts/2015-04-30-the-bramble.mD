---
layout: post
categories:
- raspberry pi
title: The Bramble
---

So far I've shown off the nitty gritty of my bramble. In this post I want to give an overview of it.

![The Bramble](/images/the-bramble.jpg)

All of the pis live on a separate 192.168.x.x than the rest of my network. This allows me to easily identify them. The entire network is under a CIDR /16 subnet so everything can talk to everything else.

The [case][case] I picked for the pi lets me stack to 7 comfortably. 7 also happens to be the number of ports on the USB hub. The ethernet switch, which has 16 ports, is also set up nicely with 7. You can fill the switch and still have room for connecting to other switches for chaining.

## Hosts

I have a few types of hosts: build, redis/memchaced, postgres, file server, and docker host. All of the hosts are labeled. If I didn't label them they would be instantly lost if I tried to find a single one.

### Build server

This server creates the docker images that are hosted in the registry also located here. Every docker host pulls from its registry. More information about the build server is located in the [Git Push with Docker][git-push-docker] post.

### Redis/Memcached

I originally had these as separate servers, but I had so much free memory that it made sense to shove them together and get another docker host out of it.

### Postgres

This server has postgres installed. I have it listening on IP and only allowing the Raspberry Pi subnet to talk to it. Backups are manual at the moment, and given the ability of the Raspberry Pi to corrupt the memory card it's top of my mind to fix the situation.

### File Server

These servers (4) are running [gluster][gluster]. They are set up to host volumes with a redundency of 2. I have tested this failure rate when one of the raspberry pis got a corrupted file system. Upon restoring a fresh install of linux I corrupted the gluster cluster almost to the point of total loss.

### Docker

There are 7 of these right now. The last 2 are empty and test beds for updating Arch. They simply have docker installed and will pull down images and run them. More information about these are located in these posts: [Docker Raspberry Pi Images][docker-rasp-pi], [nginx Docker Container][nginx-docker], and [Git Push with Docker][git-push-docker].

## Improvements

There are a few improvements I'd like to do with the bramble in a more general sense. Cable management is one of my top concerns. All of the cables I have are about 3ft long, which I had bought before the cases and new what the final setup looks like. Getting shorter cables will help make the bramble look nicer than it currently does.

I'd also like to set up a few more services to go with postgres, redis, and memcached. I think it would be cool to eventually set up elasticsearch. I have a project that currently uses regular postgres text search, but having set up elasticsearch once it would be nice to have a side project that used it.

General logging is also something I want. Right now logs are very hard to get to. There is no `heroku logs --tail`. I want to set up something like [graylog2][graylog2] or if I get elasticsearch up [logstash][logstash]. Most of the problem with getting these going is the current docker images are built for x86 not ARM. I just haven't had time to look into this yet.

[case]: http://www.amazon.com/gp/product/B00MRKSGP2
[git-push-docker]: {% post_url 2015-03-30-git-push-with-docker %}
[docker-rasp-pi]: {% post_url 2015-02-11-docker-raspberry-pi-images %}
[nginx-docker]: {% post_url 2015-03-17-nginx-docker-container %}
[gluster]: http://www.gluster.org/
[graylog2]: https://www.graylog.org/
[logstash]: http://logstash.net/
