---
layout: post
category:
- elixir
- grapevine
title: Local Clusters with epmd
description: Starting a single machine cluster with epmd.
date: 2019-03-25 8:00 PM EDT
---

For [Grapevine](https://grapevine.haus/), I recently started a single machine cluster. There are two nodes, one for the main application (sockets and Phoenix) and one as a base for telnet connections.

With this I can redeploy the main application and not disconnect players from their game sessions. This is very important as telnet connections cannot drop if they are open.

### Clustering

The start of this cluster was incredibly simple. I installed [libcluster](https://github.com/bitwalker/libcluster/) and configured each node to connect to the other via named nodes. Each node was simply their release name `@127.0.0.1` by default thanks to distillery's default configuration.

I started each node and all was well. They both connected and I was off to the races. Until I wasn't.

### Issues

This worked well until I went to deploy. I was never able to successfully deploy each node separately without one of the two not being able to connect to the other. It was very reliable which node couldn't connect. The second node to start could never connect to the other node.

I tried a few ways of getting around this but couldn't figure out what was wrong. I scratched my head for an evening before letting it sit overnight.

### Starting epmd Separately

What I think was the issue, was that epmd was being started automatically by the first node to boot. The way I was deploying was simply overwriting the entire release directory with the new code. I _think_ this screwed something up, I'm not 100% sure about this though.

By starting epmd as a separate service, the issues stopped. I set up my service files to require epmd before starting. In a sense I pushed the issue up the supervision tree!

### Service Files

This is the epmd service that Grapevine uses. Ansible places the epmd daemon at `/home/deploy/epmd` (in [this file](https://github.com/oestrich/grapevine/blob/master/deploy/roles/setup/tasks/epmd.yml).)

```
[Unit]
Description=Erlang Port Mapper Daemon
After=network.target

[Service]
User=deploy
Group=deploy
WorkingDirectory=/home/deploy/
Environment=LANG=en_US.UTF-8
ExecStart=/home/deploy/epmd
SyslogIdentifier=epmd
RemainAfterExit=no
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

And the `grapevine.service` requires that it is booted with [this line](https://github.com/oestrich/grapevine/blob/master/deploy/files/grapevine.service#L18):

```
[Install]
WantedBy=epmd.service
```

### Conclusion

If you have a local cluser, you should avoid node connection issues by starting epmd separately and not allowing for the automatic daemon start. As an extra bonus, you can now use the `epmd` binary to communicate easily with the running daemon. This lets you know each node is alive and connected, use `epmd -names` for this.

Finally, all of this code is open source so you can inspect what Grapevine does for clustering. See the [main repo here](https://github.com/oestrich/grapevine) and [the telnet node repo here](https://github.com/oestrich/grapevine-telnet).

See more about [epmd](http://erlang.org/doc/man/epmd.html).
