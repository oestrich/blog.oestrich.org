---
layout: post
category:
- elixir
- grapevine
title: Grapevine Updates for October 2019
description: Grapevine changes from October 2019
date: 2019-10-31 09:00 AM EDT
---

The last month of [Grapevine][grapevine] has started work on a new part of Grapevine we're calling Decanter. This will be a place to share news about all things text.

I am also giving one of the keynotes for [The Big Elixir](https://thebigelixir.com/) next week and have been working hard on that. I hope to see you there (or maybe on YouTube when it's uploaded!)

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

# Decanter

Decanter is a new part of Grapevine that will let you publish news about your game into a centralized place. Think of it as a new gaming news site that _only_ covers text games, the best kind of games.

It's still under construction so it's hard to show what it looks like yet. I haven't really gotten the styles in place for it to be public. But if you're extra curious, you can download the latest Grapevine code and check it out locally.

The plan is for anyone to submit a news article, and one of our editors will review it and publish the post. Posts are done in markdown and you can preview before submitting them. I still need to let you optionally attach your game to a post. If you attach your game I'll be able to display it nicely so people know what they're reading about.

We're hoping that this leads to a better landing page for news and updates than say /r/MUD. This works well, but I think we can deliver a much better landing spot for people new to MUDs and looking to learn more and see what's out there.

# Hacktoberfest

Grapevine was part of Hacktoberfest this year and I got a lot of great submissions! I was happy to see new people (or people at all!) submitting PRs and enhancing Grapevine.

Some of those include:
- An updated settings sidebar which lists all of your games, removing the table of them
- Analytics for events, so admins can see how many times they're viewed
- Generating a session token for the web client for mobile app usage
- Documentation updates and fixing broken seeds

Thanks again to everyone who submited a PR during October!

# Small Updates

- The HCL parser was renamed to UCL
- UCL parses comments now

# Social

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

[Titans of Text](https://www.titansoftext.com/) has had a lot of episodes come out. If you're in the MUD community and want to be on an episode, reach out to us! We want to have you on.

Once again, I'll be at The Big Elixir next week giving the keynote on Friday. I'll be talking about moving from an entirely stateless application to a stateful application using ExVenture and Grapevine as examples of you can achieve that. I think it's turning out to be pretty good so I hope to see you there.

# Next Month

For next month, I'm hoping to wrap up the MVP of decanter and it loose on the world. I may also play around with some web client stuff again or actually get back into ExVenture. I've been having a lot of fun extending MidMUD for NaMuBuMo and it's made me feel nostalgic for working on ExVenture again.

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
