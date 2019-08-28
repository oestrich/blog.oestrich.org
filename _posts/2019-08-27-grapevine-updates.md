---
layout: post
category:
- elixir
- grapevine
title: Grapevine Updates for June - August 2019
description: Grapevine changes from June - August 2019
date: 2019-08-27 9:00 PM EDT
---

The last few months of [Grapevine][grapevine] has been fairly slow, but I'm getting back around in the swing of things and finally feel like I have enough small things to talk about in an update post!

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

# New Game Stats

The most recent additions are new statistics for games that have MSSP or are connected to the web socket. There are three new graphs: the last 48 hours with min/max/average, the last 7 days of player counts, and the last 7 days broken up into hour of day to show the min/max/average of connected users per hour.

These are pretty neat to see with real statistics. You can see player count fluxuate as they come and go during the day and night. I'm looking at adding more historical data as well, but there is only ~8 months of data on the server so there isn't a huge use right now until more data builds up.

Below is a screen shot of the later two charts:

![Grapevine game statistics](/images/2019-08-grapevine-stats.png)

# New Web Chat Client

During June, I updated the web chat client to not suck. I'm pretty sure it was broken sometime after the merge of Gossip and Grapevine and no one noticed, me included. It is now _much_ nicer, and looks very similar to the web client for playing games.

This merges the tab interface previously in place, to show a single stream across all channels, which is fairly similar to what you'd be expecting in a game connection. You can see which channel you're chatting in with the select box next to the text area. You can swap channels by doing using a slash command to prefix your message (e.g. `/gossip hello`, to switch to the gossip channel and say hello.)

![Grapevine web chat](/images/2019-08-grapevine-web-chat.png)

# Small Tweaks

- Discord link for game profiles
- Breaking apart the data layer into a separate application
- Records chat messages for reply in the web chat
- Connect/disconnect buttons in the web client
- Hide players in the socket connection
- Nicer error for web client is enabled for signed in users only
- Save web client sessions for viewing length in the admin

# Social

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

[Titans of Text](https://www.titansoftext.com/) has had a lot of episodes come out. If you're in the MUD community and want to be on an episode, reach out to us! We want to have you on.

# Next Months

I'm still starting to slowly get back into growing Grapevine again and I don't want to jump too deep back into it to completely burn out. So I'll most likely keep doing small tasks and possibly work on some clustering stuff from before as that's pretty exciting.

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
