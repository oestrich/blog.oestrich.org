---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for February 2018
description: ExVenture changes in the month of February 2018
date: 2018-02-26 9:00AM EST
---

The last month of [ExVenture][exventure-github] was a lot of small tweaks here and there, along with a revamp of the skills system.

The documentation website is [exventure.org][exventure]. You can see the latest additions here on [MidMUD][midmud], my running instance of ExVenture. There is also a public [Trello board][trello] now.

## Skills

Skills used to be directly attached to classes. You only had what the class had. Now they are split apart from classes. You can train as many skills as you have experience for. You train them from NPCs around the world.

Skills can also be global and can still be attached to classes. When they are global or attached to a class, players will automatically train them when they become the appropriate level to unlock it. Races can also have skills now, and act in the same manner.

## Rooms

There are a few tweaks to rooms this month. They can now have features, which let players look further into details of a room. These features show up at the end of a room description and will highlight a keyword a user might `look` at to view more information.

The admin panel also had rooms spruced up, so they look more like they do in game.

![Room Admin](/images/exventure-2018-02-room-show.png)

## Mail system

The mail system lets you send mail now. You can view it in game and also via the web site. If you are offline and have an email on your account, you will receive an email notification. You will also receive an in-game notification of new mail.

![Mail view](/images/exventure-2018-02-mail-index.png)

## Handle crashes

I worked on handling crashes better. Now all of the processes that are "in" a room are linked together. When an NPC or player enters a room, they link against it. This way if any of them die, then they all die together. When they die, they load from the database to make sure they have a well known good copy of state.

Players will see a brief "Session crashed, restoring..." message and that is all they should notice (aside from losing up to 15 seconds of save data.) This works across telnet and websocket connections.

## New commands

There are several new commands available. You can `give` items to other players and NPCs. Quests can tie into this as well.

The `recall` command is new. It will return you to the zone's graveyard, but only if you have 90% of your movement left.

Lastly, there are social commands that you can add via the admin panel. They can be with or without a target.

![Socials](/images/exventure-2018-02-socials.png)

## Security

The security of the application has been increased. Telnet logins require a one time password after signing into the website.

![One Time Passwords](/images/exventure-2018-02-one-time-password.png)

When signing into the website, you can now add two factor authentication to you account.

![TOTP](/images/exventure-2018-02-totp.png)

# Small Tweaks

- Ran the elixir formatter over the codebase
- Display flags for users in the who list
- Push more events through the `notify` systems
- Stop ticking every 2 seconds, have systems react when they notice work to be done
- Send email after signing up
- Rearrange the admin sidebar
- Switch sessions to the dynamic supervisor
- Refactor the `{:user, session, user}` tuple out of the codebase
- Simple state machine for the telnet color formatter
- Keep newlines when wrapping text
- Damage is randomized, +/- 25%
- Dyanmic channels
- Web notifications for in game messaging
- `say/random` npc action, to pick a random message each tick

## Next Month

For next month, I am hoping to fill in the world detail more. I am thinking senses, lighting, and time are next. Making things more dynamic, such as damage types, is something I want to continue doing. I also need to do a refresh of the documention, some of the new features are missing and screenshots are generally out of date.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[trello]: https://trello.com/b/PFGmFWmu/exventure
