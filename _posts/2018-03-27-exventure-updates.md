---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for March 2018
description: ExVenture changes in the month of March 2018
date: 2018-03-27 9:00AM EST
---

The last month of [ExVenture][exventure-github] had a lot of _game_ improvements in addition to a lot of tweaks and bug fixes. I say game improvements because mechanics were updated in regards to leveling and gaining stat improvements.

The documentation website is [exventure.org][exventure]. You can see the latest additions here on [MidMUD][midmud], my running instance of ExVenture. There is also a public [Trello board][trello] now.

Also check out <a href="https://mudcoders.com/" target="_blank">The MUD Coder's Guild</a>, it's a slack team devoted to developing MUDs.

## Honing your stats

The biggest change (so big I reset characters on MidMUD) is being able to hone your basic stats. I got rid of your class boosting your stat a certain amount each level along with your level being added. It now increases each stat by 1 each level, and the top two stats get an extra point. The top two stats are chosen that level by the skills you used. If you use something that uses a lot of strength, your strength will go up faster.

To further customize your character, you can `hone` them. This uses your gained experience points and "spends" them on increasing a stat by 1 (or 5 if it's health/skill.) See more information on the [help for hone](https://midmud.com/help/commands/hone).

## Web Client

The web client got a very big update, commands are now clickable! As part of a semantic color update, I start sending `{command}help config{/command}` instead of `{white}help config{/white}`. This lets the web interface know that you can click those. The game can also send a different command to send when clicking a command, so you can have display text and the command itself.

A tooltip also displays:

![command tooltip](/images/exventure-2018-03-web-client-tooltip.png)

## Announcements

I added a small "blog" to the home page that lets admins post announcements about the game. You can sticky posts and write in markdown. There is an atom feed that is linked from the homepage so players can subscribe without knowing the feed URL. You can also simply not publish them to preview them on the home page as an admin.

[Here is a sample announcement.](https://midmud.com/announcements/5ede4f81-3a0a-4c36-942f-8bb15f5bff64)

## Hint system & Configuration

A hint system was introduced as a simple form of tutorial. For instance, after receiving your first tell each sign in you will see a message such as `You can reply with reply my message.` With this I introduced a player configuration system to disable these messages.

Also in the config system is disabling regeneration notices, changing your prompt, and setting the pager size. See more information [at MidMUD's help](https://midmud.com/help/commands/config).

## Say improvements

I played some of the tutorial from Discworld Mud and I was inspired to add in a say to and whisper set of commands. Whispering sends your message to only the character it is directed at and everyone else sees only that you're whispering to each other. Say to directs a message at someone in the room for all to see.

Later, I was looking over the [cheatsheet for CurryMUD](https://github.com/jasonstolaruk/CurryMUD/blob/master/cheatsheet/CurryMUD%20cheatsheet.md) and saw all the cool emote and say features it has. This inspired me to add in speaking at someone (in a cleaner fashion) and adverb phrases. I am very happy with how this feature turns out. You can be much more expressive in local chatter now. Adverb phrases also carries over to channel chat.

## Bug Fixes

Several process crashing bugs were found, luckily no player was disconnected because of them. The worst one was sending in not an integer into `quest info`. Ecto was not happy about that. Other bugs include:

- Running was missing up/down
- Display gold if only gold was in the room, no other items
- Remove command only worked for chest items
- Configuring the prompt broke sign up
- NPC combat target bug, you could trick the NPC into not attacking you when you entered a room
- Updating an NPC crashed the NPC

## Small Tweaks

- Skills have an effect whitelist now, no damaging yourself while performing a healing command
- Items have the same whitelist on a use
- Movement in/out
- Skills have a cool down
- AFK status
- Name map layers in the admin panel
- Movement should indicate direction for other players, "Guard left north", this is also clickable!
- Global broadcast when someone signs in or out
- Start of a real templating engine for messages inside the game

## Next Month

I didn't do the extra detail of the world like I had hoped last night, but I am pretty happy with what did get done instead. I think in the next month I want to have more game elements in place. Maybe data defined damage types.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[trello]: https://trello.com/b/PFGmFWmu/exventure
[mud-coders]: https://mudcoders.com/
