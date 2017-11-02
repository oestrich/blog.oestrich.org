---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Update November 2017
description: ExVenture changes in the last month
date: 2017-11-01 8:30PM
---

The last month for [ExVenture][exventure-github] has seen mostly metrics improvements and some client additions. There is also a new website for it at [exventure.org][exventure]. You can see the latest additions here on [MidMUD][midmud], my running instance of ExVenture.

## Metrics

![Grafana Dashboard](/images/exventure-nov-11-metrics.png)

I started looking into [Prometheus][prometheus] for work and started adding in metrics for various aspects of ExVenture. Some of the metrics include:

- `exventure_command_parsed_in_microseconds`
- `exventure_command_ran_in_microseconds`
- `exventure_command_bad_parse_total`
- `exventure_player_count`
- `exventure_session_total`
- `exventure_login_total`
- `exventure_login_failure_total`
- `exventure_new_character_total`

## GMCP

The server also supports GMCP now, either through normal telnet or the web client. The web client has it pushed over as a special websocket event. GMCP Modules now supported are:

- Character
- Character.Vitals
- Room.Info
- Room.Characters.Enter
- Room.Characters.Leave
- Target.Character
- Target.Clear
- Zone.Map

More information about these can be found [on the wiki][gmcp-events].

## Client

![New Client](/images/exventure-nov-11-client.png)

The updated web client uses the new GMCP pushes to allow stat bars and a side bar with room information and a mini map.

## Maps

Maps got a fairly big update in that multiple layers are supported, there are doors now, and you can go up and down. Multiple layers allow things like going under a bridge or heading up to the second floor of a house.

<img src="/images/exventure-nov-11-doors.png" width=236 height=165 />

Doors are now in place and prevent movement between rooms if they are closed, obviously. They are indicated with a `=` for closed and `/` for open states.

Finally up and down are movement directions. There isn't map indication yet, but I want to add something similar to this:

```
+---+
|[ >|
|   |
|< ]|
+---+
```

## Next Month

I want to add more in terms of NPC interaction. I started on an event system for NPCs that can really only respond to things it hears and when a character enters a room. I want to add in targetting and combat back from NPCs.

A few people have signed into the game and I get to see what they attempt to do and failed at. I think a goal every month will be to continue to smooth out things that people are attempting to do.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[prometheus]: https://prometheus.io
[gmcp]: https://www.gammon.com.au/gmcp
[gmcp-events]: https://github.com/oestrich/ex_venture/wiki/GMCP-Events
[midmud]: https://midmud.com
