---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for May 2018
description: ExVenture changes in the month of May 2018
date: 2018-05-29 9:30AM EST
---

The last month of [ExVenture][exventure-github] had a lot of development, from a [Mudlet][mudlet] package to fixing bugs in how ExVenture clusters to more NPC customization.

Links for MidMUD & ExVenture:

- [ExVenture.org][exventure]
- [MidMUD][midmud]
- [Public Trello board][trello]
- [Discord server][discord]

## Mudlet Package

[Mudlet][mudlet] is a client that can be used to connect to MidMUD or any other MUD. They have a way for the server to push a client package down to the client. This lets Mudlet be customized for MidMUD and any ExVenture server.

Through the server push mechanism, you can also auto-update the package. This lets the package stay lock-step with the server as new things are added and changed.

The package now contains and auto-mapper, stat gauges, and a tabbed chat.

<figure>
  <img src="/images/exventure-2018-05-mudlet-package.png" alt="Mudlet package" />
</figure>

## Telnet Connection Flow

To go together with the Mudlet additions, I worked on the telnet connection flow. Previously you needed to get a one time password and enter that into you chat.

I changed it up to work more like how Netflix authorizes a remote device. The telnet connection gets an ID and registers it as a session waiting to be authorized. You then click the authorize link the connection is giving you. On this page you can authorize the connection and be signed in when you switch back to your client.

This is so much more convenient than copy pasting a password and I think its just as secure.

<figure>
  <img src="/images/exventure-2018-05-telnet-authorize.png" alt="Telnet authorize" />
</figure>

## Large Scale Metrics

I spent an afternoon a few weeks ago trying to figure out how well ExVenture will scale as more and more data is pushed in and thousands of processes spin up. ExVenture performed wonderfully in my testing. I only tested the number of NPCs performing movement.

I set up roughly 14,000 rooms with 20,000 NPC processes wandering around them. This was on a quad-core Macbook Pro. The 99 percentile for movement was 350 _micro_seconds. I am still in a slight shock about how much Erlang did not care about data size.

<figure>
  <img src="/images/exventure-2018-05-movement-stabilize.png" alt="Movement stabilization" />
  <figcaption>Movement stabilizing over time</figcaption>
</figure>

<figure>
  <img src="/images/exventure-2018-05-movement-metrics.png" alt="Movement metrics" />
  <figcaption>Movement metrics</figcaption>
</figure>

## Multi-Node Bugs

As expected, turning ExVenture into a distributed app came with a significant number of bugs. Luckily I had some help on the [ExVenture Discord][discord] in finding them. I set up [Sentry][sentry] to start recording new exceptions as they came. This lets me see what was going on and record them better than a log I never look at.

A summary of the bugs I found so far:

- Channel communication was not spanning the cluster
- Raft deadlock
- Keeping players out of the game until it is online
- Rebalance zones as nodes drop out
- Current connected players were locking up
- Lots of functions were not expecting bad data back
- Strange transient data errors, I think due to processes not rebooting properly
- A room can crash often enough to tank the entire supervision tree and kill a node
- Session recovery exponential back off

Not all of these are fixed, namely the room tanking the entire tree, but most are. Getting in session recovery to be exponential is a big win. A player connected could generate thousands of exceptions by how I had it before.

## Game Jam

The [MUD Coders Guild][mud-coders] had a [Game Jam][gamejam] going on during the beginning of May. In the last moments I joined with MidMUD in the cheapest feature I could think of: a random name selection for players signing up.

This is a simple feature, but one that I am glad to get in. Picking a name has to be the hardest part of making a character, so simplifying this for new people is a good win.

## Home Page Updates

The home page now includes a web chat client! This was fun to add since it was the prototypical phoenix application. Phoenix channels are finally being used "right" in ExVenture!

The admin UI also got an update. I had been watching [Refactoring UI][refactoring-ui] and became inspired to make the admin look less crappy. There are now flash messages on all actions, and the forms should look a lot better.

If you attach an email to your account, you can trigger a password reset.

<figure>
  <img src="/images/exventure-2018-05-web-chat.png" alt="Web chat" />
</figure>

## Smaller Tweaks

- Debug command
- A basic API is available for public information
  - `curl -H "Accept: application/json" https://midmud.com`
- Admin can change web client colors
- Home page is slightly tweaked color wise
- NPCs don't target players after a respawn
- If you try using a skill that exists but you don't have, you get a nicer message for it
- Get all items in a room at once
- Delete rooms from the admin
- Handle web client disconnects, by reconnecting
- Send mail from the home page
- Disable user accounts
- Bug: Adding an item spawning did not trigger its timer
- Bug: Session recovery did not start regen
- Quests always show their progress as 100% after completion
- Bug: several bugs in the telnet protocol surrounding IAC
- Bug: web page redirects after signing in was slightly broken
- NPCs can have delayed actions
- NPCs can have mutliple actions in a single event, queued up
- Channel chatter is recorded for replay when connecting to the web client
- `scan` command to view the surrounding area
- Who list tweaks
- Commands can be marked admin only
- Automatic balancing of NPCs, change their level and the stats boost to the minimum for that level
- Continuous effect for stat boost
- Skills can default target yourself
- Target yourself via `self`
- Hide yourself when `look`ing at a room
- Format chat messages by capitalizing and adding punctuation
- Bug: NPCs targeted you before entering a room
- GMCP heartbeat message
- Lock movement when skills are cooling down
- GMCP skill status message

## Next Month

Next month I am going to continue with bug fixing while the app is in clustered mode. I have been getting some suggestions about what to tackle next so I will probably continue with that. I also wouldn't mind getting to in game forums in the next month.

I might also start switching these to a faster update cycle if I continue to have this many changes.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[trello]: https://trello.com/b/PFGmFWmu/exventure
[discord]: https://discord.gg/GPEa6dB
[mudlet]: https://www.mudlet.org/
[sentry]: https://sentry.io/
[mud-coders]: https://mudcoders.com/
[gamejam]: https://itch.io/jam/enterthemud
