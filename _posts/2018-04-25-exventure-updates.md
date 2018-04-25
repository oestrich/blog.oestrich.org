---
layout: post
categories:
- elixir
- distributed
- ex_venture
title: ExVenture Updates for April 2018
description: ExVenture changes in the month of April 2018
date: 2018-04-25 9:00AM EST
---

The last month of [ExVenture][exventure-github] had a lot of updates to clustering and some extra world details.

The documentation website is [exventure.org][exventure]. You can see the latest additions here on [MidMUD][midmud], my running instance of ExVenture. There is also a public [Trello board][trello] now.

Also check out <a href="https://mudcoders.com/" target="_blank">The MUD Coder's Guild</a>, it's a slack team devoted to developing MUDs.

## Distributed Erlang

The biggest new feature of ExVenture in the last month is the improved support for erlang node clustering. I started with adding [libcluster][libcluster] to join nodes together. Then I had the world start spanning the cluster via a really simple leader election.

Next up was using `pg2` to have the player registry and send cache updates across the cluster. The last step was using `:global` as the registration mechanism for world processes. I tried out [swarm][swarm] for this but it had some weird properties of restarting _all_ of the processes when a single one died.

You can see this step in [PR #37][multi-node-pr].

## Raft

The most recent step was implementing the leader election of [Raft][raft]. It's mostly done, but it should handle rejoining nodes and adding nodes to the cluster. Once a leader is picked, I have a set of modules that get called on the winner node. On start up this will spin out the world across the cluster.

You can see this step in [PR #39][raft-pr].

## Damage Types

Early on in the month I added custom damage types and damage resistances. Each damage type has an opposing stat that reduces the damage. This step also added an "echo back" of the damage that was actually applied. A character calculates the effects to send over, sends them, and hears back what actually happened.

You can see this in [PR #27](https://github.com/oestrich/ex_venture/pull/27) and [PR #28](https://github.com/oestrich/ex_venture/pull/28).

## Custom Colors

This is easier to show in pictures than text.

![Custom colors in the admin](https://user-images.githubusercontent.com/449228/38283850-593f2586-3786-11e8-915f-01a19fd17141.png)

![Customize your colors](https://user-images.githubusercontent.com/449228/38460721-9d8f5c4c-3a8d-11e8-8697-f7b25e4bc78c.png)

You can see this in [PR #29](https://github.com/oestrich/ex_venture/pull/29) and [PR #31](https://github.com/oestrich/ex_venture/pull/31).

## Listening

A new command was added, `listen`. This lets you tune into noises that are in the same room as you. This feature has lots of room to grow as more things make sound.

You can see this in [PR #33](https://github.com/oestrich/ex_venture/pull/33).

## NPC Status Engine

NPCs can change their status line and listen text via emotes now. As part of this they set a status key to eventually enable gating of events they do. Such as they only start combat if they are in a certain "mood" and can get out of it later on.

[PR #34](https://github.com/oestrich/ex_venture/pull/34) has this.

## Smaller Tweaks

- Admin panel tweaks for events
- Refactoring events

## Next Month

For next month I want to continue with the distributed part of ExVenture. I definitely need to get the world rebalacing as nodes fallout and re-join the cluster. I would like to continue with the other game additions from this month. Listening and the npc status engine are both very good foundations for a lot of extra features in the future.

Once I have more work in the clusting of ExVenture I want to do a deep dive blog post.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[trello]: https://trello.com/b/PFGmFWmu/exventure
[mud-coders]: https://mudcoders.com/
[libcluster]: https://github.com/bitwalker/libcluster
[swarm]: https://github.com/bitwalker/swarm
[multi-node-pr]: https://github.com/oestrich/ex_venture/pull/37
[raft]: https://raft.github.io/
[raft-pr]: https://github.com/oestrich/ex_venture/pull/39
