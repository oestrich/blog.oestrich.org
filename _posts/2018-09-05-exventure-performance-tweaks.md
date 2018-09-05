---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Performance Tweaks
description: Three steps to get more out of ExVenture and your own Elixir apps
date: 2018-09-05 9:30AM PDT
---

In the week or two leading up to [ElixirConf][elixirconf] I started trying to see how far I could push [ExVenture][exventure] with concurrent players. These are the three fairly simple tweaks I took to go from maxing out at 230 concurrent players to maxing out at 3500 concurrent players on the same machine.

Tests were performed on a desktop with a Intel Core i7-6700K and 64GB of RAM. I used [VentureBot][venture-bot] to connect to my local instance going across the local network, including wifi. I used a copy of [MidMUD][midmud] as the world.

# Single process overloaded by messages

The first time I ran VentureBot pointed at my local development version of ExVenture, I was able to get to around 230 players before room processes started falling over. The room process is a router of sorts for players in the same place of the world.

Anything that happens in a room by a player is broadcast to every other player in the room. Player processes also `call` the room to get the current state of it before acting on it through commands. The combination of these two was causing the room process to not be able to keep up with the level of messages.

Enter a side process event bus. Any notifications that the room process wants to do is pushed into this side process. This unblocks the room process from performing the relatively slow notification of characters in the room.

This pushed concurrent players up to about 600. You can see this in [Pull Request #72][pr_72].

# Single process overloaded by state size

ExVenture has a player session registry that keeps track of connected players. This is very similar to the [Elixir Registry][elixir-registry], but is a custom process on each node. The next thing that could not keep up with players was this.

When a player connects, their full user, class, skills, race, etc are all preloaded from the database. This is then pushed into the session registry. The session registry process was being slowed down by the state it held.

Almost all of this data was not required by other characters in the world. The answer then was to massively cut down what was stored in the session registry. I created a new `Character.Simple` struct that the session actually stored.

This pushed concurrent players up to about 1200. You can see this in [Pull Request #73][pr_73].

# Processes overloaded by inbox size

At this point something I really was not expecting to happen happened. I ran out of ram. Mind you this is on a desktop with 64GB of it.

The reason was almost for exactly the same reason as the previous tweak. I had fixed the registry, but the player gets pushed around a lot in messages and other process state (like the room process.)

MidMUD has about 250 rooms, which meant that there was an average of 5 players per room, but every player spawns in the same room on start. This meant that a few rooms were getting up to 30-40 players in the same room. Each action a player took was then pushed to 40 other processes. Because the player struct was preloaded to the brim it was making a _lot_ of copies of that data and blowing my RAM.

This isn't a memory leak per se, but it wasn't good.

The way around this then was to use the same simple character struct for all messages. After adding this ram usage dropped _substantially_. 1200 players took about 1GB of ram vs the previous 50GB.

This was [Pull Request #74][pr_74].

# Conclusion

This is the final guage I was able to get from Grafana on concurrent players.

![Final player count](/images/2018-09-exventure-final-player-count.png)

For reference, most MUDs these days are over the moon with anything above 20, the top one getting ~800 players. ExVenture pushed into MMO territory with these changes.

The current reason ExVenture cannot take more is because the session registry can't keep up with new players connecting which fetches the full list to see if they are already in the game. To get around this I would change up the session registry to act more like the Elixir Registry and create a worker pool to manage incoming calls.

But, there's almost no reason to try and fix this. 3500 real players is a very, very far off problem in the real world. I'll settle for 15x performance for now.

I hope these small tweaks are actionable in your own Elixir applications, especially message size between processes.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[gossip]: https://gossip.haus
[midmud]: https://midmud.com
[venture-bot]: https://github.com/oestrich/venture_bot
[pr_72]: https://github.com/oestrich/ex_venture/pull/72/files
[pr_73]: https://github.com/oestrich/ex_venture/pull/73/files
[pr_74]: https://github.com/oestrich/ex_venture/pull/74/files
[elixirconf]: https://elixirconf.com/
[elixir-registry]: https://hexdocs.pm/elixir/Registry.html
