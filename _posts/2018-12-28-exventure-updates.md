---
layout: post
category:
- elixir
- ex_venture
title: ExVenture Updates for December 2018
description: ExVenture changes in the month of December 2018
date: 2018-12-28 7:30PM EST
---

The last month of [ExVenture][exventure-github] saw a lot of internal refactors and deployment updates.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Gossip][gossip]
- [Grapevine][grapevine]
- [Squabble][squabble]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## NPC Events

Events got a huge refactor to consolidate a lot of duplicate code around actions. Previously it was not straight forward which events could have what actions, and each event implemented the action separately. This refactor sets up events and actions as full structs and modules.

Now each event lists what actions it allows. Each action is implemented once. All events also can have multiple actions in order to have more realistic NPCs.

Check out the [documentation](https://exventure.org/admin/events/) for these events.

[Pull Request.](https://github.com/oestrich/ex_venture/pull/101)

## Web Client in React

Lorecrafting in the [discord][discord] has started on a major refactor of the web client, switching over to react. It's still fairly in progress, but is moving along nicely.

## Venture Markup Language

In order to help the web client and more solidify the internal markup format, the Venture Markup Language (VML) was named. Nothing much changes other than giving it a name and actually doing parsing instead of regex replace.

There is now a set of leex and yecc parsing modules. VML now has some [documentation](https://exventure.org/admin/vml/).

[Pull Request](https://github.com/oestrich/ex_venture/pull/102)

## Deployment

ExVenture isn't the easiest thing to deploy in production at the moment, so I finally got around to working on making that easier. As part of this I finally got around to writing ansible scripts and a small deployment script to help everything out.

[See more on deploying ExVenture in the last post.](/2018/12/deploying-exventure)

## ExVenture World

Continuing with this, I have started a new hosting service called [ExVenture World][exventure-world]. This uses the ansible scripts and terraform to create new game instances in Digital Ocean very quickly. I plan on leaving this not fully automated for a while and eventually automate it fully if more people are interested in hosted games.

If you are interested in a hosted version, please let me know over on the discord channel.

## Small Tweaks

- Distillery 2.0 config provider, for everything
- Gossip design tweaks
- Use systemd
- Migrate to new servers
- Builder role
- Grapevine only login

## Social Updates

A few times people have requested a forum to be set up for ExVenture, this now exists over at [forums.exventure.org](http://forums.exventure.org).

We have a few new patrons on the [Patron][patreon], thanks for supporting!

I will be at Lonestar Elixir 2019 showing off adding prometheus metrics to your application, and Gossip will be my demo app. If you're there, make sure to say hello!

## Next Month

For the next month, I'm working on some new game mechanics, the first of which is proficiencies. I am hoping to get back around to some of the other mechanics that are already in and expand them a bit, such as items. I would like to start moving on achievements for Gossip as well.

[exventure]: https://exventure.org
[exventure-world]: https://exventure.world
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
