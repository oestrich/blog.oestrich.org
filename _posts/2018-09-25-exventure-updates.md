---
layout: post
category:
- elixir
- ex_venture
title: ExVenture Updates for September 2018
description: ExVenture changes in the month of September 2018
date: 2018-09-25 9:00AM EDT
---

The last month of [ExVenture][exventure-github] resulted in a lot bug fixes in preparation for [ElixirConf][elixirconf] with some web client features towards the end of the month.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Gossip][gossip]
- [Squabble][squabble]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Web Client

The biggest feature of the month is definitely the web client additions. Last week I was inspired by a post on `/r/mud` about playing a MUD while using less of your keyboard and more of your mouse. I've had in the back of my mind how to do this and finally got around to it.

![ExVenture web client action bar](/images/2018-09-exventure-action-bar.png)

The new additions are a target bar, that shows all of the other characters in the room and allows you to view and change your target, and an action bar that allows for up to 10 skills to be displayed. The action bar currently only auto populates with skills as you level and train new skills. The system is set up to allow for custom commands, but I haven't gotten around to configuring that part yet.

I am very happy with where it's at after a week of work. There are many updates in the future for this area, but it's a great foundation. The best part of this is you can now wander around and do combat on mobile.o

## Performance

I couldn't help myself and kept looking into performance constraints for a single server. I was able to remove a few sort of unnecessary SQL checks and a few other optimizations to unlock nearly 3x more concurrent bots on my desktop. I managed to push to 7100 bots at the same time.

![ExVenture concurrent player guage](/images/2018-09-exventure-performance.png)

I did not commit a lot of the changes for this, since it restructures some core stuff at the moment. But I did [start an issue](https://github.com/oestrich/ex_venture/issues/78) to move towards this.

What I did commit was migrating session data to an ETS table, for reads outside of the session registry process.

## Gossip

Gossip was fairly quiet last month, but did see some bug fixes. You will no longer receive heart beat messages _before_ you authenticate. There were also a few small UI tweaks that went in place.

## Translations

I started looking into translating ExVenture into other languages. Right now any text that is in a command is being run through `gettext`. This is very early work but I need to start somewhere, since I'm sure a lot needs to change depending on what language is being translated into.

I also set up a new translation site on Heroku, if anyone wants to help translate ExVenture let me know in the discord. I can get you access to that.

## Smaller Tweaks

- Save an external discord ID to push for Mudlet
- Hint system sends messages when nothing has been entered on first connect
- Rename user tuples to player tuples, moving towards many characters per account
- Help text updates
- Fix all config options being able to be set to true/false
- Cast data that goes straight to ecto (quest info #)
- Disconnect an individual user from the admin
- Admin panel has script editing for NPCs in the new react component
- Global features can have tags, to better view adding them
- Say with brackets was filtered out as an adverb phrase even if it was in the middle of your text
- `Client.Map` GMCP push on connect, hopefully letting [Mudlet pick up on an MMP map](https://github.com/Mudlet/Mudlet/issues/1962)
- Disable skills for use and view
- Custom commands for using an item, you can `drink potion`s now
- Global replacements have a read through cache, nodes dying did not repopulate the cache
- Overworld was not properly rendering newly made exits in the admin

## Social Updates

This was another big month for ExVenture and associated projects. I released [Squabble][squabble] as a separate repo and it's picked up a lot of stars after the [introduction post][squabble-intro].

There are several new patrons over at the [Patreon][patreon] as well. Welcome and thanks for supporting the project!

ExVenture itself picked up a lot of new stars this month, gaining about another 50 or so after ElixirConf. There are also a lot of new people in the [discord group][discord], many are first time learners of Elixir. Maybe I'll see you there too!

## Next Month

Given how off I usually am with guessing what takes my interest over the next month, whatever I put here will most likely be wrong. But I would like to continue with the web client, adding in configuration and keyboard short cuts. I also need to swing back around to the overworld at some point and continue fixing that up and making it fit more into the game.

I also might pull off some of the "boring" things that I've been meaning to get around to, such as adding in forums and letting each user have multiple characters. I think both would be good additions and I've been at least moving slowly towards the later.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[squabble]: https://github.com/oestrich/squabble
[gossip]: https://gossip.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/exventure
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
[squabble-intro]: https://blog.oestrich.org/2018/09/introducing-squabble
[elixirconf]: https://elixirconf.com/
[venture-bot]: https://github.com/oestrich/venture_bot
