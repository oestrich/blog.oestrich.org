---
layout: post
category:
- elixir
- ex_venture
- deployment
title: Deploying ExVenture
description: How to deploy ExVenture to a VPS
date: 2018-12-14 09:30PM EST
---

Up until recently, how to deploy ExVenture has been a bit of a mystery. I have done it a few times, for [MidMUD][midmud] most notably, but it was fairly tricky to do and entirely done by hand.

I also relied on some parts of linux like `/etc/profile.d/` to ensure environment variables were loaded, so starting the application could only be done by a login shell. It wasn't the best, but hey, it worked.

This week however, a few more people were starting to get interested in ExVenture and asking about how to host it and I finally got embarassed enough to get something more formal around how to deploy it.

I also started a hosting service called [ExVenture World][exventure-world] if anyone is interested in having a game without having to deploy it. See the announcement post for [ExVenture World on Patreon](https://www.patreon.com/posts/exventure-world-23323136).

## Vagrant

[Vagrant][vagrant] is a tool for spinning up local VMs quickly, and being able to quickly recreate them from a base box. We're going to use this to get a local install going. Install Vagrant before continuing.

```bash
vagrant up
```

After a few minutes, you'll have a local machine you can ssh into. Vagrant will provision python for us on first boot so that we can run ansible on it in our next step.

Something I've tended to have issues with on Vagrant is SSH known host issues on recreating a VM. I just learned about this snippet you can place in `~/.ssh/config` to ignore host checks. Do make sure it's only enabled for `127.0.0.1` as this is very insecure.

```
Host 127.0.0.1
  HostName 127.0.0.1
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  IdentitiesOnly yes
```

## Ansible

[Ansible][ansible] is a tool for configuring remote servers. They run playbooks to make sure a server is configured the same each time. See more information about it and how to install it on their site.

First install the roles you'll be using to provision the server. This uses Ansible Galaxy to pull down remote roles.

```bash
ansible-galaxy install -r deploy/requirements.yml
```

Next, run the setup playbook on the local machine. This will configure the local VM to be provisioned for the next few steps. Make sure to update [this line](https://github.com/oestrich/ex_venture/blob/master/deploy/group_vars/all.yml#L6) in the `deploy/group_vars/all.yml` line to include your GitHub user. It will download any keys you have associated to your account and let you into the `deploy` user.

```bash
ansible-playbook -l local deploy/setup.yml
```

## Configure

There are now two files on the machine that need to be edited, `/etc/exventure.config.exs` and `/etc/nginx/sites-enabled/exventure`. The nginx config file just needs to know what it's domain name is. The `exventure.config.exs` file needs more modification. Any empty string `""` should be filled in with the required information.

You should set up SMTP so your game can send emails, this will also require setting the from address.

The endpoint and networking lines both need to know the new domain name for your game as well. You should also connect your game to [Gossip][gossip]. It's more fun that way.

## Certbot

Once you are configured, restart nginx and then configure SSL via [certbot][certbot].

```bash
sudo systemctl restart nginx
sudo certbot --nginx
```

Follow the prompts for configuring your new domain. You should set the game to redirect any HTTP traffic to the HTTPS port.

## Deploying

Next you can deploy your game, the first deploy you should seed as well. ExVenture comes with a simple deploy script that will handle most of this for you. The last argument should be the fully qualified domain name that you previously set up. It will try SSH'ing in as deploy.

Before deploying, make sure to generate a production release with the `release.sh` script. **Note** that this needs to be run on a linux system. You cannot deploy an erlang release built on a mac to a linux remote.

```bash
./release.sh
./deploy.sh --seed my-game.example.com
```

On future deploys, you only need to use

```bash
./deploy.sh my-game.example.com
```

## A Remote Server

The previous steps got a local VM ready. Getting a remote machine is not that much harder. I recommend using [Digital Ocean](https://m.do.co/c/6e2ba5b80cde) as your VPS of choice, ExVenture World is hosted there.

Once you create a droplet (using Ubuntu 18.04), SSH in and install `python`.

```bash
sudo apt install python
```

Next setup a new host file named `deploy/host_remote` with the following:

```
[remote]
the.ip.or.domain
```

```bash
ansible-playbook -i deploy/host_remote deploy/setup.yml
```

And the rest is the same. Configure the game, configure certbot, generate a release, and deploy it.

## Conclusion

Hopefully this helps you get a remote ExVenture game set up and running. If you have any questions please come by the [Discord group][discord].

[exventure-world]: https://exventure.world/
[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[squabble]: https://github.com/oestrich/squabble
[gossip]: https://gossip.haus
[grapevine]: https://grapevine.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/exventure
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[vagrant]: https://www.vagrantup.com/
[ansible]: https://www.ansible.com/
[certbot]: https://certbot.eff.org/
