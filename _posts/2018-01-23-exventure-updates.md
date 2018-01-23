---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for January 2018
description: ExVenture changes in the month of January 2018
date: 2018-01-23 9:00AM EST
---

The last month of [ExVenture][exventure-github] had some big additions. New are NPC conversations and questing, along with other small tweaks to make NPCs feel more part of the world.

The documentation website is [exventure.org][exventure]. You can see the latest additions here on [midmud][midmud], my running instance of ExVenture.

## Stickers

I got stickers for [midmud][midmud] within the last week.

![Stickers](/images/midmud-sticker.jpg)

## NPC Conversations and Scripts

NPCs now have a script that will let players whisper to them and respond with. An example interaction is as follows:

```
You greet Guard
Guard replies, "Hello, are you here to find out about our bandit problem?"
You tell Guard, "huh?"
Guard replies, "I don't know about that, but I can help you with bandits",
You tell Guard, "yes"
Guard replies, "I heard they are hiding down in a cave."
You have a new quest.
```

You can find more about the format at [exventure.org][scripts].

## Questing

The even bigger feature is questing. NPCs can hand out quests as seen above in the example conversation. Quests replace the default NPC script if a quest is available for a player. A player doesn't necessarily get a quest when conversing with an NPC that is trying to give one out. They must hit an end of the conversation that has a `trigger: quest`.

You can see more about questing at [exventure.org][quests].

![Handing out a quest](/images/exventure-2018-01-quest-handout-1.png)

Quests currently can require items or NPCs to be killed in order to trigger them. NPCs are tracked and counted when they are killed by a player. Items are removed when turning in the quest. Quests can also be in a quest chain, requiring parents to be completed before being offered children quests. This can create a linear story that helps move a player through an area.

## NPC status line

NPCs now have a status line that displays instead of `{name} is here.` This area will also display if the NPC is a quest giver so players know to `greet` them to start a quest.

![NPC status](/images/exventure-2018-01-npc-status.png)

## Note Pad

There is a general note pad area now. Notes are simple records that have a name and a body that displays as markdown. I wanted to have something where I could record more free form ideas about the game that I could come back to later.

## Mail system

A mail system is currently being worked on, before I was distracted by questing. Right now it is read only in the game, but you can view and read mail if you have one.

## Small tweaks

- Removed a lot of passing around of the session pid
- Start running some of the modules through the elixir formatter
- Tweak NPC death
- Giving a reason for leaving or entering  a room
- NPCs have a status line and description now
- Look at items, players, npcs
- Basic session statistics are stored: how long, what commands were used
- Color codes don't for an early wrap, no longer leaving odd length lines

## Next Month

I have a few ideas on how to rework the class and skill system. I might get rid of classes as a rigid architype after reading [Designing Virtual Worlds][dvw] by Richard Bartle. He talks about having an initial class system be a starting point of skills and being free form from there. That way new players can instantly get started with minimal thinking and then later on they can completely change what they want to be. This has slowly grown on me and I really like the idea of it.

I would also like to finish the mail system.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[scripts]: https://exventure.org/admin/scripts/
[quests]: https://exventure.org/admin/quests/
[dvw]: https://en.wikipedia.org/wiki/Designing_Virtual_Worlds
