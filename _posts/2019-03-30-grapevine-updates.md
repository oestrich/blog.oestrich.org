---
layout: post
category:
- elixir
- ex_venture
- grapevine
title: Grapevine Updates for March 2019
description: Grapevine changes in the month of March 2019
date: 2019-03-30 4:00 PM EDT
---

The last month of [ExVenture][exventure-github] and [Grapevine][grapevine] was fairly light, with mostly small touch ups.

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

You can now have a custom CNAME for the web client, so it goes directly to connecting to your game. See it in action for Darkwind at [play.darkwind.org](https://play.darkwind.org).

The web client should also be faster now. It is limited to 1000 lines of history instead of _all_ of the output from the game. After being connected for an hour or so to a few games the client really dragged down. It should now keep a lot quicker as it starts dumping output.

Gauges can also undock from the bottom toolbar area and will float in the top right of the client. This can be useful for things like enemy health.

# Phoenix LiveView

For a stream I worked on making the new admin dashboard "live." It will now view any open web client as they update without having to refresh. You can watch as I add it into Graepvine from the YouTube export:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FfpRBh2kWCI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Stein

I started a new library called [Stein](https://github.com/smartlogic/stein) which handles a lot of common utilities you might need for a web application. Such as common user auth functions and storage. Stein started by pulling out common code from Grapevine, which itself was pulled from ExVenture.

Check out the documentation for Stein on [hexdocs](https://hexdocs.pm/stein/readme.html). If anything is confusing please let me know so I can update them!

# Grapevine Hosted Site

If you have a game with a web site that looks like it hasn't changed from when your game started in 1996, I started on a Grapevine hosted alternative. The plan is to have minimal customization, a blog, and the Grapevine web client built in.

If you're interested, please reach out to me on Discord or the Slack.

# Social Updates

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

This month I started a new podcast with Swift from the MUD Coders Guild called Titans of Text. It will be a bi-weekly show where we interview developers and admins from around the text gaming community. We want to reach out to more than just MUDs for this. Find us at [titansoftext.com](https://www.titansoftext.com/) and soon on iTunes.

I have also continued streaming over on Twitch every Monday at 12 PM EST doing ExVenture or Grapevine development. Join me over at [https://www.twitch.tv/smartlogictv](https://www.twitch.tv/smartlogictv).

# Next Month

I took a bit of a "break" this month and slowed down development a lot. This might continue a bit more for this month, but I want to start getting back into ExVenture and continuing with the refactor process. Pulling in Stein would be a good start, and pulling more out of ExVenture into Stein as well. I also plan to keep tweaking the web client and getting some kind of customization for that in place, letting you change the font and font size to start with.

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
