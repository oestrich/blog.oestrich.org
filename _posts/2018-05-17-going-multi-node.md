---
layout: post
categories:
- elixir
- distributed
- ex_venture
title: Going Multi-Node with Elixir
description: How I took ExVenture from a single node to many nodes
date: 2018-05-17 9:30AM EST
---

From the [last update of ExVenture][april-update], ExVenture can now be configured to run in a multiple node configuration. In this post I'll show you the basics of how I did that.

## What is ExVenture?

ExVenture is a multiplayer text-based game, a [Multi-User Dungeon (MUD)][mud-wikipedia], server.

You can see more information about ExVenture at [exventure.org][exventure] and [GitHub][exventure-github]. You can see it running on [MidMUD][midmud]. There is also a public [Trello board][trello].

## Starting point

ExVenture was heavily geared towards running as a single node to start out. I used [Registry][registry] very heavily, which only works for a single node. I also have a few [local cache processes][items-cache] that out of the gate wouldn't work well when spanning multiple nodes.

My first thoughts where about splitting up the app into an umbrella app and figuring out how to boot the web on one node, the telnet connection on another, etc. I started talking about this in the [MUD Coders Guild][mud-coders], and I got a question of "why?"

This was a great question and got me thinking about what else I could do.

I eventually settled on trying to get the same application booting on all nodes and a leader node starting the processes that can only exist once in the cluster, e.g. the world.

## Clustering

First up was connecting up nodes in an automated fashion. Since my app was heavily geared towards being a single node, this should be fine. Each node wouldn't talk to each other and the entire world would be spun up on each node.

This was extremely simple with [libcluster][libcluster]. It was as simple as installing the hex package and adding this to my configuration files:

```elixir
config :libcluster,
  topologies: [
    local: [
      strategy: Cluster.Strategy.Epmd,
      config: [hosts: [:"world1@host", :"world2@host"]]
    ]
  ]
```

Then when starting my app I switched to booting with `iex` to get the `sname` flag available.

```bash
iex --sname world1 -S mix
iex --sname world2 -S mix
```

## Picking a Leader

Once the world was clustered, I started on picking a leader. I had heard about [Raft][raft] before, but never really looked into it.

If you are interested in clustering at all, I'd highly recommend giving the paper a read. It is very simple to follow along and understand. Which was the main point of creating Raft, a simple to understand consensus algorithm.

For ExVenture, I went with implementing my own Raft module because I only wanted the leader election part of Raft. I don't need (at least yet!) the rest of Raft.

You can see that [in the Raft module][raft-module-github]. There is a lot to this module that doesn't need to be covered here, but it boils down to the group picks a leader and that leader calls the [subscriptions][raft-subscriptions] that care about who was leader.

## A leader is picked

Once a leader is picked, the [`Game.World.Master`][world-master] process uses [pg2][pg2] to find all of the other `Game.World.Master` processes and sees what zones are alive in the cluster.

After finding out what nodes are alive it spins up the zones not online across the cluster, using a [simple rebalacing][zone-rebalance] algorithm.

## Global process registry

I also switched to using the global process registry as part of this. I started looking at [swarm][swarm] but it seemed to be something different than what I was looking for.

I will most likely end up changing this in the future but switching `{:via, Registry, {Game.NPC, id}}` to `{:global, {Game.NPC, id}}` was an extremely simple change that worked. So I went with it and haven't looked back.

## Messages spanning the cluster

The game was now officially multi-node and you could play on either `iex` servers and see other players and NPCs on the other one.

There was only one final step and that was setting up `pg2` groups for each of my caches and my communication layer.

This is very simple and can be done as follows in the init function and a slight change to casting to your GenServers.

```elixir
@key :items

def init(_) do
  :ok = :pg2.create(@key)
  :ok = :pg2.join(@key, self())

  #...
end

def insert(item) do
  members = :pg2.get_members(@key)

  Enum.map(members, fn member ->
    GenServer.call(member, {:insert, item})
  end)
end
```

## Next Steps

With this up and running I was able to get [MidMUD][midmud] in a multi-node (3 world servers) set up for production. It has been working out pretty well so far and only once I found a large bug (of not being able to communicate across nodes in channels.)

I will post more updates as I continue enhancing the distributed nature of ExVenture.

You can see everything described here in these two pull requests, [#37](https://github.com/oestrich/ex_venture/pull/37) and [#39](https://github.com/oestrich/ex_venture/pull/39).

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[trello]: https://trello.com/b/PFGmFWmu/exventure
[april-update]: /2018/04/exventure-updates/
[mud-wikipedia]: https://en.wikipedia.org/wiki/MUD
[registry]: https://hexdocs.pm/elixir/Registry.html
[items-cache]: https://github.com/oestrich/ex_venture/blob/master/lib/game/items.ex
[mud-coders]: https://mudcoders.com/
[libcluster]: https://github.com/bitwalker/libcluster
[raft]: https://raft.github.io/
[raft-module-github]: https://github.com/oestrich/ex_venture/blob/master/lib/raft.ex
[raft-subscriptions]: https://github.com/oestrich/ex_venture/blob/master/lib/raft/server.ex#L12
[world-master]: https://github.com/oestrich/ex_venture/blob/master/lib/game/world/master.ex
[zone-controller]: https://github.com/oestrich/ex_venture/blob/master/lib/game/world/zone_controller.ex
[pg2]: http://erlang.org/doc/man/pg2.html
[zone-rebalance]: https://github.com/oestrich/ex_venture/blob/11790b258509117965ab1f22fd24993c2f4f4767/lib/game/world/master.ex#L73-L91
