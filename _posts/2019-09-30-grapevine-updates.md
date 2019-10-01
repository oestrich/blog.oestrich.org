---
layout: post
category:
- elixir
- grapevine
title: Grapevine Updates for September 2019
description: Grapevine changes from September 2019
date: 2019-09-30 10:00 PM EDT
---

The last month of [Grapevine][grapevine] has picked up pace, I've been working through a lot of little things here and there to get back into the development groove.

If you're new please check out the [ExVenture & Grapevine Patreon][patreon] page and consider supporting to help pay for servers or get your own hosted instance of ExVenture!

Links for ExVenture & Grapevine:

- [ExVenture.org][exventure]
- [Grapevine][grapevine]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Patreon][patreon]
- [ExVenture Trello board][exventure-trello]
- [Grapevine Trello board][grapevine-trello]
- [Discord server][discord]

# Mobile Client

I started on a mobile client for Grapevine. This uses react native and connects similarly as the web client. This means it will proxy everything through my servers. I am going to head this route because it will be significantly less development to have a web client and a mobile client going if they use the same telnet backing code.

I haven't spent a ton of time of it yet, but after a few hours I have it connecting, displaying, and sending text to a real game. It's pretty encouraging.

<figure>
  <a href="/images/2019-grapevine-mobile-client-start.png" target="_blank">
    <img alt="Grapevine's mobile client" src="/images/2019-grapevine-mobile-client-start.png" height="400" />
  </a>
</figure>

# Hosted Sites

I spent a bit of time working on the settings for a hosted site. I mostly wanted to figure out how to allow for safe markdown display based on input from anyone. I got this working by finding a library that will strip text down to only markdown allowed tags.

[See Spigot's hosted site](https://spigot.grapevine.haus).

<figure>
  <img alt="Spigot's hosted site" src="/images/2019-grapevine-hosted-site.png" />
</figure>

# IO Lists

After ElixirConf this year I switched Spigot to using IO lists instead of strings. This got a pretty great speed boost, which you can read more about on the [SmartLogic Blog](https://blog.smartlogic.io/elixir-performance-using-io-data-lists/).

# Small Tweaks

- Receive an alert anytime your game has a failed scrape
- Admin panel for channels
- Fix bug when creating a new game and adding an image
- gossip-elixir is on latest telemetry
- API for the games page
- Record timestamp of activity through the web client
- Social fields on a game
- Generic events
- List current events on the homepage
- Display the play button even when signed out
- Rate limit channel sends

# Social

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

[Titans of Text](https://www.titansoftext.com/) has had a lot of episodes come out. If you're in the MUD community and want to be on an episode, reach out to us! We want to have you on.

# Next Months

Next month is [NaMuBuMo](https://namubumo.com), so I will hopefully be working on a world for that. I've been playing around with a file based format for running a game, as a twist to ExVenture. We'll see if it sticks, but I like where it's headed.

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
