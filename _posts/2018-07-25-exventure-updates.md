---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for July 2018
description: ExVenture changes in the month of July 2018
date: 2018-07-25 9:30AM EST
---

The last month of [ExVenture][exventure-github] has been pretty sparse compared to previous months. This has been due to me starting a new side service called [Gossip][gossip]. Gossip is a cross game chat network for MUDs.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Gossip][gossip]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Gossip

Gossip has been rolling around in my head for a while while working on ExVenture, but there has always been more pressing matters. About a month ago we were talking on the MUD Coders Guild about cross game chat and that inspired me to kick this off.

Gossip is similar to the [I3 Network](http://lpmuds.net/intermud.html) except it uses more standardized technologies. Secure WebSockets are the transport layer and all events are in JSON. You can see the [documentation on Gossip](https://gossip.haus/docs).

## ExVenture Remote Channels

The first big feature of Gossip is remote channels. You can flag a channel as a Gossip channel and it will try to send all communications up to the network.

## ExVenture Remote Player Status

When your game is configured for Gossip, all signed in players are pushed up to Gossip. This lets other connected games see your players sign in and out. Right now all notifications are displayed to users on ExVenture. This is an optional but highly suggested feature for games that are not based on ExVenture.

This should help make your game feel more alive by letting your small pool (maybe just 1!) of players see others on the network.

The local `who` list will also display remote players, so users can see who is on the network.

<figure>
  <img src="/images/2018-07-exventure-remote-player-list.png" alt="Remote player list" />
</figure>

## ExVenture Remote Tells

The final big feature for Gossip right now is remote tells. If you're syncing your local players up to Gossip (and ExVenture does) then remote games and send tells to your local players. ExVenture lets you initiate tells and also handles `reply` for remote players.

This degrades nicely for remote games that do not support tells. As part of connecting a Gossip client says what features they support. For games that are not built on ExVenture remote tells are optional, but highly suggested.

<figure>
  <img alt="First set of remote tells" src="/images/2018-07-exventure-first-remote-tells.png" />
</figure>

## Gossip Games List

I definitely suggest you check out the [Gossip Games](https://gossip.haus/games) list. So far we have 5 games connected, more seem to be checking out Gossip every day!

## Gossip Clients

The Gossip client from ExVenture has been pulled out into it's own [Elixir hex package](https://hex.pm/packages/gossip). If you're developing your own Elixir MUD (and there are a lot of you out there) then come join the fun and add the package.

Right now you do need to implement the full set of callbacks for all of the features of Gossip, but I would like to let you dictate which features your game supports and slowly add them in.

There is also a [Ranvier bundle](https://github.com/oestrich/gossip-ranvier) that supports remote channels and player status updates.

## Next Month

Next month I hope to start back on ExVenture more and leave Gossip to sit for a bit. I think Gossip has a good enough feature set to let other games implement what's there and get some more feedback from the community.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[gossip]: https://gossip.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/midmud
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
