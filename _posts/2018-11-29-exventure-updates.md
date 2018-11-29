---
layout: post
category:
- elixir
- ex_venture
title: ExVenture Updates for November 2018
description: ExVenture changes in the month of November 2018
date: 2018-11-28 10:30AM EDT
---

The last month of [ExVenture][exventure-github] has been pretty good. ExVenture was on an Elixir podcast and did an introduction meetup with UtahElixir. There were also a lot of Gossip related updates.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Gossip][gossip]
- [Grapevine][grapevine]
- [Squabble][squabble]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Podcasts

I was on the [Elixir Mix](https://devchat.tv/elixir-mix/emx-027-eric-oestrich/) podcast, episode 27. On it, I talked about all of the different projects that ExVenture encompasses. I also went into some of the topics I covered in the Going Multi-Node ElixirConf talk I gave. Definitely give it a listen!

I also talked with UtahElixir about ExVenture and introduced it to the group. Mark Ericksen is an organizer for UtahElixir and wants to use ExVenture as a way of teaching various elixir topics and wants to go over various parts of it over the next few meetups. How cool!

## Multiple Characters

I finally wrapped up the split apart from a single user account, to a user account that has characters. This was a fairly long process as it touched a lot of pieces of the code. I have been slowly going through everything so as to not feel like it was a suffocating refactor and the work finally paid off.

I was very happy with how easy this final refactor ended up being after the positioning ExVenture for this with previous refactors.

You can see this in [Pull Request #90](https://github.com/oestrich/ex_venture/pull/90).

## Grapvine Login

I have been talking with the MUD Coders Guild for a what's next thing for Grapevine and we came up with the idea of Grapvine being an OAuth provider. I have previously written one so I figured it'd be fun to do again. With multiple characters in place, now was the time to add it into ExVenture as well.

I couldn't use existing OAuth code because they ship with their own backing tables, and I wanted to reuse the games table. If you update ExVenture and are connected to Gossip, you now support Grapevine authentication. It takes some small extra configuration on Gossip's end to get redirect URIs in place, but otherwise that's it.

You can see this in [Pull Request #91](https://github.com/oestrich/ex_venture/pull/91) and [Pull Request #4](https://github.com/oestrich/grapevine/pull/4).

## Gossip

Gossip has gotten some work as well. I rewrote the underlying sync code to push around versioned payloads. This lets me capture deletes in addition to creates and updates.

With this in place, I was able to add in in-game events. You can add events to your game and they will be displayed on Grapevine. I would like to get a nice calendar view for this, but I went with simple until some data starts getting added in.

Gossip and Grapevine also got some minor styling tweaks. The homepages for each should better explain what they are.

Gossip also tracks player count over time now and generates nice looking charts. This is viewable in [Grapevine](https://grapevine.haus/games/MidMUD).

![Grapevine Player Counts](/images/2018-11-gossip-player-counts.png)

## Gossip Refactors

The Gossip server and clients both got major refactors to their socket code. I was able to do a cool event router of sorts for the server. This can be seen [here](https://github.com/oestrich/gossip/blob/master/lib/web/socket/router.ex).

The elixir client for Gossip was also updated and recieved a 1.0 version. I blogged about this recently, [check it out](/2018/11/writing-evented-websocket-client).

## Small Tweaks

- Grapevine - display game connection status
- Refactor `Game.Format` into smaller sub modules
- Fix the overworld map editor to allow for adding exits
- Gossip - delay sending game offline messages
- Grapevine - password resets
- Update raisin deps
- Gossip client - telemetry events
- Gossip - migrate to telemetry
- Gossip - require players to be online to send tells
- Social emotes now actually broadcast to the room you're in

## Social Updates

This month was pretty good for ExVenture on the social front. Lots of new stars across all of the projects!

I went to [The Big Elixir](https://www.thebigelixir.com/) earlier this month and gave my Going Multi-Node talk again. I met a lot of new people who were previously into MUDs or thought they seemed cool and are new to MUDs. Welcome everyone I met from The Big Elixir.

There are several new patrons over at the [Patreon][patreon] as well. Welcome and thanks for supporting the project!

The [discord group][discord] has several new members in the last month.

## Next Month

I am planning on refactoring the event system behind NPCs next. I am currently scoping out what I would like it to look like so this should happen. It will be a big undertaking but I think well worth the effort.

I am still looking at achievements as well. Progress is slow but it is happening. I am still trying to figure out some sync issues with them but they should be do-able. With the new sync in place for Gossip -> Grapevine these should be pretty easy to get going as well.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[squabble]: https://github.com/oestrich/squabble
[gossip]: https://gossip.haus
[grapevine]: https://grapevine.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/exventure
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
[squabble-intro]: https://blog.oestrich.org/2018/09/introducing-squabble
[elixirconf]: https://elixirconf.com/
[venture-bot]: https://github.com/oestrich/venture_bot
[gossip-backbone]: https://github.com/oestrich/gossip-backbone
[raisin]: https://github.com/oestrich/raisin
