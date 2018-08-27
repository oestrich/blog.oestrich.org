---
layout: post
category:
- elixir
- ex_venture
title: ExVenture Updates for August 2018
description: ExVenture changes in the month of August 2018
date: 2018-08-27 9:30AM EDT
---

The last month of [ExVenture][exventure-github] kicked back in action now that [Gossip][gossip] is mostly stable. I tried to stick to a general theme of bug fixes and world building additions though. I wanted to get [MidMUD][midmud] read in advance of [ElixirConf][elixirconf].

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Gossip][gossip]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Gossip

Just because I let Gossip "sit", doesn't mean I didn't work on it! There were a few minor features that got touched on.

The home page now features a random connected game to be highlighted. Any game that is connected and has a home page url might show up on the front page.

![Gossip Homepage](/images/2018-08-exventure-gossip-homepage.png)

Gossip also got a cleaned up README since a lot of people mistakenly thought you needed to install NodeJS to connect. This was on the README as a requirement, but as a requirement to start the server itself.

There is a [media page](https://gossip.haus/media) that contains a footer you can add to your homepage if you're apart of the network and want to show it off.

## Gossip Elixir Client

The Elixir client got some updates as well. There were a few bugs hanging out related to the player list. If a game went offline completely with players attached, they would never go away from your games list. The list gets sweeped regularly for games that haven't been seen in a while.

[Version 0.5.0 is out now on hex.pm.](https://hex.pm/packages/gossip)

## Multi-Node Bugs

There were a few multi-node bugs that got cleaned out in preparation for my ElixirConf talk. I wanted to make sure that this was very stable before getting on stage to talk about it. The raft leader selection had a few small bugs in it. If a node went offline zone rebalancing wasn't actually happening (which is bad.)

Everytime a node came back online, a new election was triggered, no matter what. This one wasn't a horrible bug per se, but having the leader force itself should have been enough.

The world leader also now checks to see if all of the zones are online now periodically. I had noticed once that MidMUD had a zone down after a node died. I don't really know what happened, but scanning for all zones being online shouldn't be a bad thing.

I also found a slight issue with using `:pg2` as a world leader list. Occasionally `:pg2` hadn't caught up with the node dying before the world leader was trying to rebalance. Which resulted in the leader calling a dead node. I got around this by first filtering out the members list against the connected node list.

## Performance Enhancements

I was curious about the possible performance of MidMUD during ElixirConf. Of course it will be a huge hit and _everyone_ will be signing into it, so I wanted to make sure it could stand up to the brunt of that.

It's a good thing I did look into this as before any changes ExVenture completely fell over at about 230 connected players. After a few tweaks I was able to push this to 1200 connected players before it fell over due to RAM usage (how crazy is that.)

I will expand upon the changes I did in a future blog post. I don't want to give it all away in this!

Until then, you can checkout ou [Venture Bot][venture-bot] which is the bot framework I set up to connect as a player to ExVenture.

![ExVenture 1200 players](/images/2018-08-exventure-1200-players.png)

## General UX Improvements

No one could figure out how to complete a quest, which is my bad. I updated the hint system to let the player know whenever a quest is completable and also displays exactly _how_ to complete that quest.

> HINT: You have a quest that you can complete, use {command}quest complete [quest_id]{/command}. See {command}help quests{/command} for more information.

In addition to completing quests, picking them up was also a tad confusing. The hint message for NPCs talking to you was also updated. This will display only once per sign in.

> HINT: You got a tell from an NPC. This might be the lead up to a quest. Please read carefully what they are asking about and you can {command}reply{/command} with your response.

With any luck these two hint additions should make questing much more accessible.

## Templating Enhancements

The templating system got a decent set of additions. There is now a `context` struct that is very similar to a Phoenix `conn` struct. You can pipe it through a set of assigns and then finally "render" it through a template string. This makes it nicer to read in the code and I think removes some complications in calling `template`.

A few extra variables are available in room descriptions, such as the name of the room and the zone.

Every NPC, Item, Room, and Zone are also available to template in a lot of game strings via a new global resource template system. I was referring to a lot of these resources in quests and room descriptions on MidMUD and finally got annoyed enough while constantly renaming things to fix the problem.

Now you can refer to any of those 4 resources with `[[room:1]]` in game strings (the admin will say if it's available) and the templating system will find room 1 and print its name.

Room descriptions can also template room features in specific spots now. The feature key is available as a template item. This lets you weave in the features into specific spots of the description. All features will still be appended to the end of the description if none are used inside the text.

## Script Additions

The final "big" thing for the month was an addition to quest scripts. You can now mark a line as triggering to another line with a delay. This lets you break up huge blocks of text with multiple tells, eventually leading to a line that has listeners or triggers a quest.

This was fun to add as the outcome was much more realistic in chatting with an NPC.

## Smaller Tweaks

- Global room features
- Telnet login link is a link
- Admin displays inline help for quest steps
- Runs could parse poorly and cause a crash
- Duplicate users in the who list due to issues in Session.Registry
- Players going AFK at the login prompt could show as signed in
- Channel command by itself was bugged
- Gossip: API to view currently connected games
- Multiple message of the days and after sign in messages
- Updating many depdencies, we're on Elixir 1.7
- Disable skills so they don't display or are usable
- Strip colors from notification text
- Add any flag to users, to add "Patron" text to patrons
- Older saves did not migrate cleanly when some stats were no longer defaulting

## Social Updates

This was a pretty big month for ExVenture and Gossip. The cowboy websockets blog post was picked up a few places and a lot of people found out about both. The discord server has gotten a few new people, some of which are new to Elixir and looking forward to learning it through ExVenture which is great!

If you're interested in joining the server to talk about ExVenture, [this is the Discord invite][discord].

I also have stickers if anyone spots me at ElixirConf next week.

![ExVenture and Gossip Stickers](/images/2018-08-exventure-stickers.jpg)

## Next Month

With ElixirConf over after the first week of September, I might get back to bigger features. I would like to split characters apart from users, which is a huge refactor. But a refactor that has been waiting for a while. Some of this move was started with the tweaks that pushed ExVenture to 1200 players.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[gossip]: https://gossip.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/midmud
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
[elixirconf]: https://elixirconf.com/
[venture-bot]: https://github.com/oestrich/venture_bot
