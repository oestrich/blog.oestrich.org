---
layout: post
categories:
- elixir
- gossip
- grapevine
title: Gossip Sync Websocket Protocol
description: Gossip has a new sync protocol on its websocket enabling microservices via websockets
date: 2018-10-16 9:00 EDT
---

In the last few weeks I started on a new project that is tied to [Gossip][gossip], the chat network for text-based games (aka MUDs.) This project is called [Grapevine][grapevine]. Grapevine is a player site where you can register your characters across any game that is connected to Gossip.

Links for Gossip & Grapevine:

- [Gossip][gossip-github]
- [Gossip Elixir Client][gossip-elixir]
- [Grapevine][grapevine-github]
- [Raisin][raisin-github]
- [Backbone Sync][backbone-sync]

### Sync Protocol

For Grapevine, I wanted to have it connect with the standard [Elixir client for Gosisp][gossip-elixir] which meant it needed to live on the same socket as a standard game. I went with adding in a Gossip network-level application that pretends to be a game for most parts, but when one connects it gives the socket super powers.

The biggest of those super powers at the moment is recieving `sync/*` events. When the socket connects it gets a set of sync events for all of the channels and games that Gossip knows about. Each event is bundled up into 10 channels or games, to help out a little bit with network overhead.

A sync event looks similar to this:

```json
{
  "event": "sync/channels",
  "payload": {
    "channels": [
      {"id": 1, "name": "gossip", "description": "...", "hidden": false}
    ]
  }
}
```

When Grapevine receives this it creates or updates a local version of that remote record. I called this the backbone sync, you can view it [here on Gossip's side](https://github.com/oestrich/gossip/blob/master/lib/web/socket/backbone.ex).

These events also trigger for any creates or updates while the socket is connected. When a new game is made or a game is edited, [Gossip broadcasts an internal sync event](https://github.com/oestrich/gossip/blob/master/lib/gossip/games.ex#L94) that the socket is subscribed to. This immediately pushes out and keeps the remote applications up to date always.

### Raisin

One of the main reasons for wanting to head this socket route, is that I also wanted other higher privileged applications to live on the network. The next one being [Raisin][raisin-github], which will be a moderation tool for the network.

With the start of Raisin, I wanted to reuse the same sync code across both projects. I had previously done this for Rails applications via Rails Engines, but I wasn't sure how to accomplish this. After talking with some fine folks over at the [MUD Coders Guild][mud-coders], I think I stumbled across something that works.

I pulled out all of the common code into a [new library][backbone-sync] that Grapevine and Raisin depend on. This library hosts its own ecto repo so it can internally save the data. Configuration wise, it's the same database as Grapevine or Raisin. That lets you use the backbone internal schemas outside of the backbone library.

I'll have more to say about this as I continue to explore it.

### Conclusion

It is extremely exciting to watch these sync events propagate. The closest I've seen to this are webhooks, but being connnected to a duplexed socket is another thing entirely.

I hope you all check out the code backing all of this, and maybe even decide to join the network yourself as a player!

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[gossip]: https://gossip.haus
[gossip-github]: https://github.com/oestrich/gossip
[gossip-elixir]: https://github.com/oestrich/gossip-elixir
[grapevine]: https://grapevine.haus
[grapevine-github]: https://github.com/oestrich/grapevine
[raisin-github]: https://github.com/oestrich/raisin
[backbone-sync]: https://github.com/oestrich/gossip-backbone
[patreon]: https://www.patreon.com/exventure
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
