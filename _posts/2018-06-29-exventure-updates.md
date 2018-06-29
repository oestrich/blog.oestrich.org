---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for June 2018
description: ExVenture changes in the month of June 2018
date: 2018-06-29 9:30AM EST
---

The last month of [ExVenture][exventure-github] was mostly the overworld, which I [posted about previously][overworld]. There were a lot of other smaller tweaks too.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

## Overworld

The overworld is a new extremely large feature for ExVenture. Admins can "paint" an overhead map that players can wander around. Exits can be made to rooms in the "old world" for travel back and forth.

I was very happy to get this working, even if there is still a large [TODO list][overworld-todo].

<figure>
  <img src="/images/2018-06-exventure-overworld-movement-exits.gif" alt="Overworld movement" />
</figure>

## Gossip

There was some conversation this week on the [MUD Coders Slack][mud-coders] that made me finally get around to a cross game chat network. I started the [Gossip][gossip] project for this. It's a simple Phoenix app that has a cowboy websocket handler to get around not using Phoenix channels.

I went with this route so others didn't have implement the Phoenix channels protocol in other languages. I also wanted something that was extremely stable, since Phoenix Channels has bumped versions fairly recently I don't want to have my library bump and break any existing connected game.

So far its been a fun exercise and it's currently working in a simple form of routing messages to connected applciations. I am in [the middle of an ExVenture connect][gossip-pr].

## Mudlet Additions

When connecting via Mudlet, the package down auto downloads the world map. This should save a lot of wandering time for players to get a fully functional mapper.

I also got a [patch][mudlet-autowrap] into Mudlet based on some functionality I pulled out of the ExVenture package. This is available as part of Mudlet 3.10.

Mudlet has been working on Discord integration, and ExVenture is ready whenever that ships. I have basic status setting functionality set up at the moment.

## Admin React

To sort of build myself up to what I thought I would need for the overworld, I started adding some nicer interfaces to the admin panel via React components. The first two I did were effects and quest scripts.

Effects are now done via a nice form that dumps to raw JSON. No longer do you need to edit that huge textarea by hand!

Quest scripts are now filled out by a nice form instead of of a text area and I also have a script tester right in the panel. You can see how the NPC would react without having to head in game.

<figure>
  <img src="/images/exventure-2018-06-script-editor.png" alt="Script Editor for Admins" />
</figure>

## Exits

Room exits got a massive overhaul in preparation for the overworld (and continued afterwards.) They are now one way, but the admin panel makes and destroys both directions. This drastically simplified the usage of exits inside of the code base.

The only tricky part was getting the reverse exit to be created for the overworld, but I will take one tricky spot over it being littered throughout the whole app.

As a bonus, this let me finally add in diagonal movement. You can go `north east` now.

## Character Stats

I revamped the character stats. Movement points are gone, replaced with Endurance Points. They aren't used yet, but I would really like to let skills use Skill Points or Endurance Points. I might also rename Skill Points sometime in the future to better reflect the split. I want Endurance Points to be more physical based, and Skill Points to be more "mind" based.

The basic attributes have also been reworked. The new six attributes are:

- Strength
- Agility
- Intelligence
- Awareness
- Vitality
- Willpower

They all go in pairs of two to boost the HP, SP, and EP. Strength and Agility boost HP, Intelligence and Awareness boost SP, and Vitality and Willpower boost EP. This feels a lot better than how it was previously and has at least 100% more thought put into what they are and how they're named.

## Web Client

The web client got a major face lift over the weekend. It now has a sidebar chat to track channel messages, so they don't get lost in the flow. I also tweaked the bottom bar to feel better, paddings and margins are now similar, along with font and font size. Some panels also got squeezed and moved around. Hopefully the new client looks much nicer to you as it does to me.

<figure>
  <img src="/images/exventure-2018-06-web-client.png" alt="Web Client" />
</figure>

## Next Month

I hope to continue with the overworld and Gossip integration. These are both big features that will be fun to continue with in the coming months.

I am in the process of reading _The Art of Game Design_ by Jesse Schell so I expect some changes to come from that. Namely I want to start thinking of how I want people to experience the game and start tweaking it to better reflect that. I expect this to be a slow process but highly worth the thought put into it.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/midmud
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
[overworld]: https://blog.oestrich.org/2018/06/exventure-overworld
[overworld-todo]: https://github.com/oestrich/ex_venture/issues/63
[gossip]: https://github.com/oestrich/gossip
[mudlet-autowrap]: https://github.com/Mudlet/Mudlet/pull/1708
[gossip-pr]: https://github.com/oestrich/ex_venture/pull/68
