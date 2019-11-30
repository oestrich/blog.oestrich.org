---
layout: post
category:
- elixir
- grapevine
title: Grapevine Updates for November 2019
description: Grapevine changes from November 2019
date: 2019-11-30 05:00 PM EST
---

The last month of [Grapevine][grapevine] continued background "unseen" changes. The big deal for this month was splitting off the socket code to its own Erlang node. This lets me redeploy the website without taking down the chat socket.

I also rebranded the Patreon to be aimed at myself vs any of the individual projects. I think this makes sense going forward since I have a handful of projects all in the MUD space but separate as entities; Grapevine, ExVenture, and Titans of Text. The new patreon is at [patreon.com/ericoestrich][patreon].

Links for ExVenture & Grapevine:

- [ExVenture.org][exventure]
- [Grapevine][grapevine]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Patreon][patreon]
- [ExVenture Trello board][exventure-trello]
- [Grapevine Trello board][grapevine-trello]
- [Discord server][discord]

# New Socket Node

The main new big feature is the web chat socket is now living on its own Erlang node. What this means is when I deploy the website, the socket won't have to go down. This means chat should get disconnected much less often. It will still go down if I need to deploy the socket code, but that will be much less than the website.

Some day I might get adventurous enough to get hot code loading working so it never goes down. To get here, let's make Grapevine hoppin'.

# Spam Users

Grapevine for a while has been getting some spam accounts being created, which hasn't been that bad but after a few months the users have finally gotten to the point where I needed to do something about it. Mostly to prevent my mail reputation from going down the drain.

To prevent spam going forward I added recaptcha to the sign up. This is just the simple "I am not a robot" checkmark. I also cleared out any account that looked like a spam account from my unverified email list. I hopefully didn't delete anyone's real account, but clearing out 30+ pages of fake emails is a mind numbing thing.

# New Games

We have a new set of games showing up on the chat, Schism from ChatMUD made a MOO client for chatting so a few more MOOs have shown up and the chat is active (finally!)

It seems like Grapevine is starting to reach a critical mass of games and players to be active throughout the day. Which is really exciting.

# Decanter

I'm still moving on getting Decanter set up but since I was fairly busy last month prepping for a conference talk (a keynote for The Big Elixir) it got side tracked. I'm hoping we can have this ready to go by the start of next year.

For those who don't know Decanter is going to be a place to post game news and updates. Think a gaming news site but specifically for us.

# Social Updates

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

[Titans of Text](https://www.titansoftext.com/) has had a lot of episodes come out. If you're in the MUD community and want to be on an episode, reach out to us! We want to have you on.

Early this month I gave one of the keynotes for The Big Elixir. The video of this should be up soon. I think it went very well and I got to show off how Grapevine and ExVenture worked internally. I will also be giving this talk at Lonestar Elixir in February, maybe I'll see you there!

# Next Month

I was previously going to swing back around to the web client, but now that the chat is being used I might work on some admin tools for that. They're sorely lacking at the moment. I also noticed a lot of you all trying to use emotes and complaining that they didn't work well 😃, so figuring something out for that will be good.

[exventure]: https://exventure.org
[exventure-world]: https://exventure.world
[exventure-github]: https://github.com/oestrich/ex_venture
[grapevine]: https://grapevine.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/ericoestrich
[exventure-trello]: https://trello.com/b/PFGmFWmu/exventure
[grapevine-trello]: https://trello.com/b/bWZ00VpS/grapevine
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
