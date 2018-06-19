---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Overworld
description: An overview of the newest feature for ExVenture, the overworld
date: 2018-06-19 9:30AM EST
---

This week I wrapped up the absolute basics of ExVenture's largest new feature, the overworld.

<figure>
  <img src="/images/2018-06-exventure-overworld.png" alt="Overworld overview" />
</figure>

This was a huge feature that's barely been scratched.

## Map Editor

First up was I wanted to make a nice way to edit the overworld in the browser. This was my first big React app. You can see it in action below:

<figure>
  <img src="/images/2018-06-exventure-map-editor.gif" alt="Overworld map editing" />
</figure>

I have a lot of ideas on how to make this nicer, but the editor works for now. Things I want to add are: a nicer color picker, which will display the color instead of just using the word; and some way to paint bucket large areas with symbols and colors.

I may also set up a terrain swatch to bundle together a symbol and color (and eventually cell attributes) into a nice picker.

## Exit Editor

After the map is created, you need to hook it up with rooms so players can move in and out of the map. This is another react page that does live API updates.

<figure>
  <img src="/images/2018-06-exventure-creating-an-exit.gif" alt="Overworld exit editor" />
</figure>

## Moving Around

Once exits are created, players will be able to move around freely between the room based zones and the overworld based zones.

Here is movement on [MidMUD][midmud], going from the room based town of Milay to the countryside around it:

<figure>
  <img src="/images/2018-06-exventure-overworld-movement-exits.gif" alt="Overworld movement" />
</figure>

## Backing Architecture

Overworld zones are set up as a pool of [Sector][sector] processes. Each sector handles a portion of the grid of cells in a map, 10x10 right now. The calling or casting process [determines which sector](https://github.com/oestrich/ex_venture/blob/master/lib/game/environment.ex#L62) to talk to so this should scale well.

Only minor updates for the commands were needed to get the overworld working. Mostly the updates were stripping functionality and making sure they did less on the overworld. Commands like involving items in particular were broken on the overworld, because I didn't want to implement item storage on the overworld quite yet.

## TODOs

I've started a [GitHub issue][todos] to track all the things that are still missing. The big ones right now are:

- Items in the overworld
- NPCs in the overworld
- Say communication should go further than your current cell
- Seeing other players on the map

I am looking forward to seeing the overworld slowly fill out now that the basics of being able to get to it and move around are done. It is a huge relief to have this part over.

## Links for MidMUD & ExVenture

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Patreon][patreon]
- [Public Trello board][trello]
- [Discord server][discord]

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/midmud
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[sector]: https://github.com/oestrich/ex_venture/blob/master/lib/game/overworld/sector.ex
[todos]: https://github.com/oestrich/ex_venture/issues/63
