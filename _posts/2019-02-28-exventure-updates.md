---
layout: post
category:
- elixir
- ex_venture
- grapevine
title: ExVenture & Grapevine Updates for February 2019
description: ExVenture & Grapevine changes in the month of February 2019
date: 2019-02-28 8:00AM EST
---

The last month of [ExVenture][exventure-github] and [Grapevine][grapevine] saw some refactors in ExVenture and a brand new web client in Grapevine.

Links for ExVenture & Grapevine:

- [ExVenture.org][exventure]
- [Grapevine][grapevine]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Patreon][patreon]
- [ExVenture Trello board][exventure-trello]
- [Grapevine Trello board][grapevine-trello]
- [Discord server][discord]

# Grapevine Web Client

<figure>
  <img alt="Grapevine Web Client" src="/images/2019-02-grapevine-webclient.png" />
</figure>

The biggest new feature of Grapevine, and the thing that has consumed most of my time is a new web client for accessing telnet based games. This feature is turned on by the game administrator and configured by them for all users. This way a consistent and simple interface is given to everyone, keeping configuration on the player non-existant at the moment.

Game admins can configure only what gauges should show at the bottom of the client right now. I'm slowly working on more options, but this is a good start. It's powered by [GMCP](https://www.gammon.com.au/gmcp), a telnet protocol extension.

The client is powered by React and Redux, and even has a few reducer tests. This is my first go around with a "real" react and redux application and I'm liking it. It finally "clicked" as to why I'd want to add in redux this time.

I even set it up so games can have a subdomain on their own domain that launches directly to the web client. If you have a game and want to enable this, let me know!

# Grapevine Clustering

In order to enable the web client to not drop connections on a deploy, I pulled a node that holds the connection to games out of the main node. This hosts just the telnet connections and that's about it.

This lets me deploy the main Grapevine code as often as I like and not have it drop connections. Users still see a brief period of web client <-> Grapevine disconnection but they eventually reconnect and pick right back up.

I need to work on better messaging around this, but the core works well.

[Check out the new telnet only application.](https://github.com/oestrich/grapevine-telnet)

# ExVenture Character Tuple

Not to forget that ExVenture exists, I started work on a fairly large refactor that tackles something that's been annoying me for a while. The character target system passed around tagged tuples that tag what kind of character the tuple is, e.g. `{:player, player}`.

This is really ugly to read in the code so I've been working through the code base removing those in favor of pushing around the plain `Character.Simple` struct, adding in a type field when its necessary to distinguish between the two. I'm hoping that this will also let me collapse the Room process caring about the two as separate entities.

[See the current state of this pull request on GitHub.](https://github.com/oestrich/ex_venture/pull/129)

# Social Updates

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

I will be at Lonestar Elixir at the end of this week, giving a talk about monitoring your application with Prometheus.

I have also continued streaming over on Twitch every Monday at 12 PM EST doing ExVenture or Grapevine development. Join me over at [https://www.twitch.tv/smartlogictv](https://www.twitch.tv/smartlogictv).

Finally, SmartLogic launched a new podcast centered around Elixir in Production. [Find it over at podcast.smartlogic.io](https://podcast.smartlogic.io/)

# Next Month

Next month I will continue expanding the features that the web client can do. I feel like it's pretty close to being the right amount of simple and usable. I don't want to provide that many options, as MUD clients that are extremely powerful [already exist](https://www.mudlet.org/). I also want to get back to ExVenture and keep that pushing forward, but we'll see where the wind takes me.

[exventure]: https://exventure.org
[exventure-world]: https://exventure.world
[exventure-github]: https://github.com/oestrich/ex_venture
[grapevine]: https://grapevine.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/exventure
[exventure-trello]: https://trello.com/b/PFGmFWmu/exventure
[grapevine-trello]: https://trello.com/b/bWZ00VpS/grapevine
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
[squabble-intro]: https://blog.oestrich.org/2018/09/introducing-squabble
[elixirconf]: https://elixirconf.com/
[venture-bot]: https://github.com/oestrich/venture_bot
[gossip-backbone]: https://github.com/oestrich/gossip-backbone
[raisin]: https://github.com/oestrich/raisin
