---
layout: post
category:
- elixir
- ex_venture
title: ExVenture Updates for January 2019
description: ExVenture changes in the month of January 2019
date: 2019-01-25 9:30AM EST
---

The last month of [ExVenture][exventure-github] saw a continuation of refactors and some exciting changes in Gossip & Grapevine.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Grapevine][grapevine]
- [Squabble][squabble]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Gossip

Gossip got a few nice improvements. I added the ability to scrape telnet based games that aren't hooked up to the chat netowrk. There is a protocol called MUD Server Stats Protocol (MSSP) that probes telnet connections for game information. Gossip now sweeps every hour for compatible games, adding to the list of online games.

Achievements are also on the move. You can add achievements to your game and have them display. I haven't been working on unlocking them yet, but the core is there. I was delayed on unlocking them because of the weird split in Grapevine and Gossip.

## Grapevine

Speaking of that, Gossip and Grapevine have merged into just Grapevine. I added all of the features to Gossip, and then renamed Gossip to Grapevine. This split was sort of awkward and made it confusing for people signing up. It was also harder than it needed to be to sync new stuff from Gossip to Grapevine. With this in place it should be much easier to add new features.

Continuing with that, I've been working on the design for Grapevine. There's a new home page that displays online games with their new cover art. This makes it look more familiar to other gaming stores, even if Grapevine isn't a store.

## Proficiencies

I've also been working on some new game features. I've been working on proficiencies which is a way of gating things behind a rank. An example might be a Swimming proficiency, which you can require 5 ranks in in order to move between two rooms. If you have 4 ranks or under, you are prevented from moving through. Players can hone ranks by spending experience points.

I think this is a nice simple mechanic that can be used in interesting ways. No proficiencies are bundled into ExVenture, you only have the ones you design for your game. You can watch me work on honing proficiencies on this [YouTube video](https://www.youtube.com/watch?v=S8xDg8kKDi4).

## CI & Test refactors

I worked on setting up CI again for ExVenture (and Grapevine.) As part of this I worked a lot on refactoring the test suite to remove global state Agents. Instead I started sending messages to the test process as code interacts with remote processes. It was pretty nice to get this working and also dramatically stabalized the test runs.

Also as part of CI, I'm building a new release inside of Docker and pushing to S3. I don't have a nice URL for these yet, but the top commit on the `master` branch should have a built release you can use to deploy. I want to figure out how to generate a clean URL for easy use. I would like to do a blog about how this works come next month.

## Small Tweaks

- Player URL is case insenitive on grapevine
- Display channels that a game is connected to on gossip
- ExVenture (un)subscribes to Gossip channel updates
- Lots of new metrics
- Refactor Game.Help
- Continuing migrating formatting calls to the new template system
- Wrap calls to injected modules (as module attributes) in a single module
- Help updates across the cluster
- Password resets for Gossip accounts
- Work continues on the react web client

## Social Updates

I'll be at Lonestar Elixir at the end of next month talking about getting metrics from your application with Prometheus. I hope to see you there!

I've been streaming over on Twitch every Monday at 12 PM EST doing ExVenture or Grapevine development. Join me over at [https://www.twitch.tv/smartlogictv](https://www.twitch.tv/smartlogictv).

## Next Month

Next Month I would like to continue expanding what proficiencies can influence. I think it would be cool to get them in front of items and a few other things. I will also keep pushing forward on the new Grapevine. I'm looking at adding a fairly simple web client to Grapevine to easily connect to telnet games, among other things.

[exventure]: https://exventure.org
[exventure-world]: https://exventure.world
[exventure-github]: https://github.com/oestrich/ex_venture
[squabble]: https://github.com/oestrich/squabble
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
