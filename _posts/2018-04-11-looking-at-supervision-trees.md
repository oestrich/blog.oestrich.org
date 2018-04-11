---
layout: post
categories:
- elixir
- ex_venture
title: Looking at ExVenture's Supervision Tree
description: Taking a deep look at ExVenture's supervision tree
image: /images/exventure-supervision-tree.jpg
date: 2018-04-11 9:00AM EST
---

I was watching [The Hitchhiker's Guide to the Unexpected][elixir-youtube] (YouTube link) by Fred Hebert and in that there is a neat excercise of writing out your supervision tree on a whiteboard and seeing how things would fail. With this you could better determine what happens to your application as things go wrong.

I decided this would be a good excercise to do on [ExVenture][exventure-github]. This is a fairly long post that goes through the full supervision tree for ExVenture.

You can see ExVenture in action on [MidMUD][midmud].

## Supervision Tree

<a href="/images/exventure-supervision-tree.jpg" target="_blank">
![ExVenture Supervision Tree](/images/exventure-supervision-tree.jpg)
</a>

This is the supervision tree that ExVenture ships with now. There are roughly 3 levels in the photo.

## First Level

This is the top level directly underneath the application. It contains, in start up order:

- `Data.Repo` - the Ecto repo
- `Web.Supervisor` - the Phoenix supervisor
- `Game.Registries` - a collection of `Registry`s
- `Game.Supervisor` - a the top level supervisor of the game
- A ranch listener is also started at this level, but it spins off into the ranch application

At this level the supervision strategy is `rest_for_one`. This is fine because if the Repo dies the rest of the app should be rebooted, something went wrong. As we'll find later on the loads process with an ID to fetch from the database to ensure a clean state is fetched on process restarts (if something crashes.)

## Second Level - Web.Supervisor

This supervisor is mostly sitting on top of the Phoenix `Endpoint` along with a few process monitors for the `TelnetChannel` and a [Cachex][cachex] cache. It is handled by a `one_for_one` strategy. This is fine as none of them are really connected to the other, this supervision level is mostly to break sections up for my benefit.

## Second Level - Game.Supervisor

This supervisor contains the "world" along with supporting processes. In start up order:

- `Game.Config` - an agent that caches game configuration
- `Game.Caches` - a supervisor of Cachex caches along with GenServer processes that are related to caching
- `Game.Server` - a tiny process that used to do more, but now keeps player telemetry up to date
- `Game.Session.Supervisor` - the supervisor for player sessions
- `Game.Channel` - a gen server that tracks player sessions and which channels they are joined to, inspired by Phoenix Channels
- `Game.World` - the supervisor that supervises the game world, see more below
- `Game.Insight` - a small GenServer that tracks bad command parsing
- `Game.Help.Agent` - an agent that load internal game help

This level has `one_for_one` as its strategy. At this level most sub-trees are fairly separate and can handle rebooting (to my knowledge) without interfering with other sub-trees.

## Third Level - Game.World

This is the heart of the app. It contains everything the user interacts with in the game. Its direct children are `Zone.Supervisor` supervisors. This level has a strategy of `one_for_one`. This is fine because each zone is self contained and can reboot on its own.

### Zone.Supervisor

This level has in startup order:

- `Game.Zone` - the zone's state, which tracks what rooms/npcs/shops are online
- `Game.Room.Supervisor` - A supervisor of rooms that belong to the zone
- `Game.NPC.Supervisor` - A supervisor of NPCs that belong to the zone
- `Game.Shop.Supervisor` - A supervisor of shops that belong to the zone

The reboot here is `one_for_all`. If any of these processes die something bad happened and the whole zone should restart. To further go into this, the Zone process tracks processes inside the sibling supervisors and if that dies then the rest should go as well. If the supervisors at this level died something _really_ bad beneath them happened and the rest should be restarted.

When the sibling supervisors start they are started with the zone id. With this they figure out which children should be loaded at boot. Tese supervisors start processes as `transient` because they may be terminated normally and should not be rebooted, e.g. if someone deletes a spawner for an NPC then the process will be terminated cleanly.

The sibling supervisors are also a `one_for_one` strategy. This is fine as each process under them are fairly self contained and separated mostly for programmer benefit, this could probably be a big bag of processes directly under the `Zone.Supervisor`.

## Take Aways

While doing this I was able to rework some of the tree. I pushed `Game.Config` further up the tree since that seems important. I also pushed more GenServers into the `Cache` sub-tree since they were similar.

One of the other reasons I did this was to figure out how to split up the app on separate nodes. This excercise taught me that it's currently not as easy as I was hoping. I figure the `Web` tree could be pulled off without doing much of anything, yet I found out that the `Game` tree is connected in a few spots that prevent it from immediately being pulled off. This would have been an annoying lesson to learn as I did that, now I know before hand and can fix the problems I found first.

In going multi-node, each of the first level would be good as a separate OTP app in an umbrella app. I had previously started with that but the application was too new for that to be useful. If I split them up again, I can boot nodes that are just for web, just for telnet connections, or just the world. I think this is a next step for going multinode.

I hope this was useful reading through seeing why I picked what I did and also finding out I had a few things ordered wrong. I hope you go through your own apps and try out a similar excercise on them.

[elixir-youtube]: https://www.youtube.com/watch?v=W0BR_tWZChQ
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[cachex]: https://github.com/whitfin/cachex
