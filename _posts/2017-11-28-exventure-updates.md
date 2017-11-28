---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for November 2017
description: ExVenture changes in the month of November 2017
date: 2017-11-28 9:00AM
---

The last month for [ExVenture][exventure-github] has seen a lot of NPC changes and lots of documentation. The documentation website is [exventure.org][exventure]. You can see the latest additions here on [MidMUD][midmud], my running instance of ExVenture.

## NPC Events

I have expanded a lot on the event system for NPCs. You can see a full list [at exventure.org][docs-npc-events].

![NPCs wandering around, and triggering aggro](/images/exventure-nov-17-npc-wander-fight.png)

The most exciting thing that happened was NPCs can now fight back! They will also wander around near where they spawned and aggro players if they stumble into any. This is all stored and edited in a huge JSON array right now, but I would like to eventually make it a nice interface. This is the bandit's events on MidMUD:

```json
[
  {
    "type": "room/entered",
    "action": {
      "type": "target"
    }
  },
  {
    "type": "combat/tick",
    "action": {
      "weight": 10,
      "type": "target/effects",
      "text": "{user} slashes at you.",
      "effects": [
        {
          "type": "slashing",
          "kind": "damage",
          "amount": 2
        }
      ],
      "delay": 2.0
    }
  },
  {
    "type": "tick",
    "action": {
      "type": "move",
      "max_distance": 2,
      "chance": 25
    }
  }
]
```

## Documentation

I have finished a first pass at the documentation site. All of the large systems in the admin panel have documentation at this point. There are still a few small things with rooms and NPCs that need documentation.

## Item instances

Items now create instances, and are tracked as such. This allows extra data to hang out with the item id. Right now only `created_at` is included with the `id`.

## Minor Tweaks

A few minor tweaks that make life nicer:

- Registration can be via the web
- Registration will display errors if any come up
- Commands parse a little better, backend code could be deduplicated
- Common mistakes such as `kill target` and point users towards the help system
- Set a target when using a skill, `magic missile target`
- Ask for optional email, and allow registering via the home page

## Next Month

Next month I'd really like to continue with combat. Right now there is a static amount of damage you will do, it might be nice to do a range of damage. I also want to add in some kind of dodge mechanic. Items also don't affect combat at all either.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[docs-npc-events]: https://exventure.org/admin/events/
