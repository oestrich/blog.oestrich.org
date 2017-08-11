---
layout: post
categories:
- elixir
- erlang
title: Using GenServer :via
description: How to create your own :via module
date: 2017-08-11 9:30AM
---

For a [side project][ex_venture] I wanted to figure out how to use `:via` with GenServer. I have two different GenServers that should respond to similar GenServer interface, [NPCs][npcs] and [User Sessions][sessions].

I went this route because I already had two [registries][registry] going, one for each type of instance. The NPCs registry was a standard unique registry and the session registry was a duplicate registry so I could easily determine which sessions were online and had a connected user.

With that in place it was very easy to get a `:via` module set up. I did not have to handle two of the required functions for `:via` which deal with registering and unregistering.

## :via whereis_name

The first thing I had to get going was finding the correct PID given an existing registry. For NPCs I simply delegated off to the elixir registry since it was already built in. I patterned matched against a tuple to determine the split.

```elixir
def whereis_name({:npc, id}) do
  Registry.whereis_name({Game.NPC.Registry, id})
end
def whereis_name({:user, id}) do
  player = Session.Registry.connected_players
  |> Enum.find(&(elem(&1, 1).id == id))

  case player do
    {pid, _} -> pid
    _ -> :undefined
  end
end
```

## :via send

This was a similar instance, delegate off to the standard registry for NPCs and find the pid for user sessions.

```elixir
def send({:npc, id}, message) do
  Registry.send({Game.NPC.Registry, id}, message)
end
def send({:user, id}, message) do
  case whereis_name({:user, id}) do
    :undefined ->
      {:badarg, { {:user, id}, message} }
    pid ->
      Kernel.send(pid, message)
      pid
  end
end
```

## Conclusion

It was fairly easy to get this going and now I can send a message to either set of GenServers by only know their database ids. I can start to hide the knowledge of if the server is a session or an NPC pid going forward for things that won't care.

[ex_venture]: https://github.com/oestrich/ex_venture
[npcs]: https://github.com/oestrich/ex_venture/blob/master/lib/game/npc.ex
[sessions]: https://github.com/oestrich/ex_venture/blob/master/lib/game/session.ex
[registry]: https://github.com/oestrich/ex_venture/blob/master/lib/game/registries.ex
