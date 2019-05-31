---
layout: post
category:
- elixir
- grapevine
title: Grapevine Updates for May 2019
description: Grapevine changes in the month of May 2019
date: 2019-04-30 8:00 AM EDT
---

The last month of [ExVenture][exventure-github] and [Grapevine][grapevine] continued the pace of last month, mostly continuing work on the web client and test server.

Links for ExVenture & Grapevine:

- [ExVenture.org][exventure]
- [Grapevine][grapevine]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Patreon][patreon]
- [ExVenture Trello board][exventure-trello]
- [Grapevine Trello board][grapevine-trello]
- [Discord server][discord]

# Modals

New to the web client are modals. These are sent as a known GMCP message that opens up a modal. The text is processed as ANSI escape codes for coloring and acts as a draggable terminal for now.

There are more plans in the future but I got it working at all and kept moving on. I would like to make this be stylable from the game, to enable some pretty cool flows. Letting users have a wizard of prompts, receiving input, button presses, display images, etc. Lots of cool things can be done here and done fairly generically from the game's perspective, so another client like Mudlet can have a package that does the same.

<figure>
  <img alt="Grapevine web client with modals" src="/images/2019-05-grapevine-webclient-modals.png" />
</figure>

# Color Processing

I completely rewrote the color processing part of the web client. Before I was using [Anser](https://github.com/IonicaBizau/anser) and in a fairly bad way. Partially sent escape codes didn't process properly and I wasn't merging the buffer after processing it. It was a stop gap that worked well enough to get where we're at.

The rewrite is a fully custom parser utilitizing small bits from Anser (namely the really ugly regex to parse the escape code properly.) Past that it handles parse errors and merging new text in to update the last line. All of the lines except for the last line are considered "sealed" and that lets react not have to re-render anything once it renders. This keeps the parse and DOM updates to a minimal.

So far this new parser is very fast, sticking to about 0.3ms for each step. I am pretty happy with this and may pull it out into its own package.

# ExVenture

Movement is starting on ExVenture again. I fixed a few bugs for a new patron that picked the hosting tier. While fixing those bugs, the itch to work on ExVenture again popped up. I am starting to plan out some refactors I can do to help bring in some ideas from Spigot into the main project.

I'm hopeful that Grapevine will slow down and I can get back to some minor refactors.

# Small Tweaks

- Grapevine, Send the user's real IP if enabled for a game
- Grapevine, Load speech synthesis voices and display, starting on text to speech
- Grapevine, Detail list for events
- ExVenture, Fix some bugs around Character.Simple
- ExVenture, Fix changing your password

# Social

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

[Titans of Text](https://www.titansoftext.com/) has had two more episodes come out. We talked with Griatch from Evennia and Janey from Stillborn. These were both great episodes and we have some really exciting ones lined up. It's pretty great to have a fairly large backlog of people to talk to.

I have also continued streaming over on Twitch every Monday at 12 PM EST doing ExVenture or Grapevine development. Join me over at [https://www.twitch.tv/smartlogictv](https://www.twitch.tv/smartlogictv).

I am also going to be one of the keynote speakers for [The Big Elixir](https://thebigelixir.com) this year, talking about ExVenture. I am very excited for this and I hope to see you there!

# Next Month

Next month I'm thinking I'll start working on the chat side of Grapevine, making that nicer and more in line with the web client. More advanced features of the web client may be on hold for a brief period, but I may also get pulled back in as this is the part of Grapevine that people use. I would like to start some small refactors of ExVenture as well.

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
