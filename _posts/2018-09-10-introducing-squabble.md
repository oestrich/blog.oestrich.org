---
layout: post
categories:
- elixir
- raft
- squabble
- ex_venture
title: Introducing Squabble, Simple Leadership Election
description: Squabble is a simple leadership election package using parts of the Raft Protocol
date: 2018-09-10 9:30AM EDT
---

As part of my ElixirConf talk, I showed off [Squabble][squabble] my new Elixir package for selecting a leader in your cluster. Squabble uses the leadership election part of the [Raft Protocol][raft]. This was pulled out of [ExVenture][exventure], my clustering text based multi-player game server.

## What is this?

Squabble takes the leadership election part of Raft and plugs it into your Erlang/Elixir nodes. Each node has a single Squabble process that communicates with the other Squabble processes across the cluster.

Together they vote for a leader and track the other nodes coming and going. Whenever a leader is selected, a callback module you provide is called on the new leader node. This lets you do a single set of work across the cluster.

## Why do I want this?

ExVenture uses this to rebalance the stateful virtual world. The world is broken up into zones (think of a whole city as a zone) and they can be pushed across the cluster. When a node goes down it takes down part of the world. The leader notices this and then detects which zones are not alive across the cluster and spins them up again.

## Why not use the Raft package?

This is separate from the [Raft Elixir package][raft-elixir], it is trying to do something different. I want a simple system that only picks a leader node in my cluster. I don't need a log or anything else that comes with the Raft protocol.

This is not intended as a replacement for the Raft elixir package.

## How do I use this?

Setting up Squabble is pretty simple.  First  You add it to your supervision tree _after_ your application is clustered. You can cluster your nodes with [libcluster][libcluster].

This is the setup [ExVenture](https://github.com/oestrich/ex_venture/blob/ecfa6cd/lib/ex_venture/application.ex#L19) uses.

```elixir
children = [
  {Cluster.Supervisor, [topologies, [name: ExVenture.ClusterSupervisor]]},
  {Squabble, [subscriptions: [Game.World.Master], size: @cluster_size]},
  # ...
]

Supervisor.start_link(children, opts)
```

When the leader is selected, the [World.Master][world-master] is invoked to rebalance the cluster and start the world. Any a node dies, the `World.Master` will see what is no longer alive in the cluster and start those processes again.

```elixir
defmodule Game.World.Master do
  @behaviour Squabble.Leader

  @impl true
  def leader_selected(term) do
    GenServer.cast(__MODULE__, :rebalance_zones)
  end

  def handle_cast(:rebalance_zones, state) do
    rebalance_zones()
  end

  defp rebalance_zones() do
    # finds alive zones and determines which are not started,
    # and starts them
  end
end
```

## Conclusion

I hope you give Squabble a look. It doesn't fit all needs, but for a simple set of callbacks in a small cluster it works very well. Check it out and help make [Squabble][squabble] even better!

[squabble]: https://github.com/oestrich/squabble
[raft]: https://raft.github.io/
[exventure]: http://exventure.org
[raft-elixir]: https://github.com/toniqsystems/raft
[libcluster]: https://github.com/bitwalker/libcluster
[world-master]: https://github.com/oestrich/ex_venture/blob/ecfa6cd6a21045c0bed7efe78058fafae2535405/lib/game/world/master.ex
