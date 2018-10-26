---
layout: post
category:
- elixir
- ex_venture
title: ExVenture Updates for October 2018
description: ExVenture changes in the month of October 2018
date: 2018-10-26 8:45AM EDT
---

The last month of [ExVenture][exventure-github] was a lot of minor refactors and other chores that needed to happen plus a migration back to working on [Gossip][gossip].

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Gossip][gossip]
- [Grapevine][grapevine]
- [Squabble][squabble]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Gossip

I started working on Gossip again. There is a new support flag that you can support, which loads basic information about the games that are online, `games`. You can also receive events for when a game connects and disconnects now. This should help you keep local caches more up to date. See the [Gossip Docs](https://gossip.haus/docs) for more information about the new events.

Gossip also got a HUGE refactor of it's socket code. It had started getting fairly spaghetti like so I spent a weekend refactoring it to be much nicer, [this pull request shows how that went](https://github.com/oestrich/gossip/pull/32).

## Grapevine

I also launched a new site called [Grapevine][grapevine] that is for players on the Gossip network. [Games have a profile page now](https://grapevine.haus/games/MidMUD), which shows the basic game information plus current players and how to connect and play yourself.

[Players also have a profile](https://grapevine.haus/eric) which shows the in game characters you've registered. Grapevine is a full Gossip application which means you can send a tell to it. This allows grapevine to know who your player is and can link the character to your profile.

There is a lot of work on this site that I want to do in the future. The biggest of which will be achievements. See [this issue on Gossip's GitHub](https://github.com/oestrich/gossip/issues/33) if you're interested in helping shape what these look like.

## Gossip Elixir Client

The Gossip elixir client also got a major refactor thats about to wrap up. It was getting large enough to finally have some patterns take root and be able to refactor to be nicer (it might have also been drifting towards spaghetti code as well oops.) [This pull request shows this refactor in action](https://github.com/oestrich/gossip-elixir/pull/8).

The client also got updated to include all of the current features that Gossip the server supports. You can also not support every feature that Gossip and the client can handle by providing only the callback modules that you wish to support. I need to update the docs for this but expect a 1.0 client soon!

## Raisin

Games supporting Gossip is starting to have a minor upswing and I want to be ready for when this takes off. To help with that I started [Raisin][raisin] to handle moderation. Raisin is a network application like Grapevine and logs the goings on of the Gossip network. This is all it does at the moment, but this is the place where network admins will be able to moderate the community.

## Backbone Sync

To support Grapevine and Raisin, Gossip needed a new level of connecting game which I've called a network application. This application gets socket super powers when it connects and starts to receive sync events. When Grapevine loads it gets a set of `sync/games` and `sync/channels` events, and will continue to get them as anything changes.

To keep code duplication minimal, when I started Raisin I pulled out the common sync code into a separate OTP application. It manages its own repo and keeps sync code in one spot. [gosisp-backbone][gossip-backbone] is up on GitHub.

## Small Tweaks

- API representers to handle outputting in various formats
- Remote erlang QR code library for an elixir version
- Player saves get updated via a common function
- Create a character struct for a user, continuing the migration to multiple characters per account
- Gossip update indexes to use the SQL `lower()` function
- Gossip can have user agent repo links
- Gossip blocks certain game names
- Gossip can have connections for your game
- Gossip's your game page got a spruce up

## Social Updates

This month was pretty good for ExVenture on the social front. Lots of new stars across all of the projects!

There are several new patrons over at the [Patreon][patreon] as well. Welcome and thanks for supporting the project!

The [discord group][discord] has several new members in the last month.

I was also promoted to be an admin of [The MUD Coders Guild][mud-coders] this month. I look forward to helping make Gossip and co a more official part of the guild. It was already unofficially official since there is a `#guild-outreach` channel where we all talked about new features, but now we can take it to bigger and better heights!

## Next Month

Over the next month I will most likely keep going with Gosisp, Grapevine, and Raisin. In particularly Raisin needs to be able to have some power on the network. That will be fun to write the events for.

I also want to keep moving on achievements. I will most likely add them into ExVenture first, before starting on Gossip itself. I would like to be ahead of the network for Gossip instead of letting ExVenture play catch up.

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
