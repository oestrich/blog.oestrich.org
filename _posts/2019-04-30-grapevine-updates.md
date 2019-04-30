---
layout: post
category:
- elixir
- grapevine
title: Grapevine Updates for April 2019
description: Grapevine changes in the month of April 2019
date: 2019-04-30 8:00 AM EDT
---

The last month of [ExVenture][exventure-github] and [Grapevine][grapevine] continued the pace of last month, mostly being touch ups, and the addition of a new test server.

Links for ExVenture & Grapevine:

- [ExVenture.org][exventure]
- [Grapevine][grapevine]
- [ExVenture World][exventure-world]
- [MidMUD][midmud]
- [Patreon][patreon]
- [ExVenture Trello board][exventure-trello]
- [Grapevine Trello board][grapevine-trello]
- [Discord server][discord]

# Spigot

I started on a new test server to easily send new GMCP messages without needing to modify ExVenture to properly send the events I am looking to act on in Grapevine.

I am also using this as an opportunity to better layout a game server using some better principles learned with more Elixir development time. Commands now act as controllers, keeping light and mostly rendering "views" back to the telnet session. I'm playing around with a better layout for internal processes as well. I'm pretty happy with the direction this is headed, and I might do a deeper blog post soon.

# OAuth & Telnet

As part of Spigot, I finally got an initial dream for Grapevine working. You can click "Play" in Grapevine and be signed into a MUD using OAuth of sorts. It takes the standard OAuth flow and shoves it into the telnet IAC flow. TLS is required for this to work in order to keep things more secure.

Sample flow:

```
Server: IAC DO OAUTH
Client: IAC WILL OAUTH
Client: IAC SB OAUTH Start {host: "grapevine.haus"} IAC SE
Server: IAC SB OAUTH AuthorizationRequest {response_type: "code", client_id: "...", 
  scope: "...", state: "..."} IAC SE
Client: Requests confirmation from the user, displays a standard OAuth request asking
  for scopes and the connection
User: Approves request
Client: IAC SB OAUTH AuthorizationGrant {state: "same as above", code: "..."} IAC SE

Server then goes through standard OAuth
```

See the current draft spec [on a gist](https://gist.github.com/oestrich/1fc022a448c36ac2aa5a099f4ca3778c).

# Web Client

I was able to get around to letting you pick the font and font size for the web client. It doesn't save yet, so each session you need to tweak it. I got part of the way through saving preferences and then was side tracked and haven't gone back yet.

I am getting very close to starting to add cool things to the web client, such as modals, sound, and text to speech. I keep stalling in Spigot, trying to get it to a nice spot. Its in a good enough place that I can go back to working on the reason it exists, extending Grapevine!

# Small Tweaks

- Event listing page
- Homepage featured games
- About page
- Site map
- Contact form
- LiveView for concurrent player counts
- Discourse SSO
- Invalid byte in the JSON stream
- Web client play button logic fix
- Socket connections should enable the player graph
- TLS cert pinning for self-signed certs
- Langing page after registration

# Social

We have a few new patrons this month, welcome! Thanks for supporting both ExVenture and Grapevine. If you would like to support both projects, [check out the Patreon.][patreon]

[Titans of Text](https://www.titansoftext.com/) has had two more episodes come out. We talked with DarkWind and Sindome. A few more episodes are already lined up, they should be pretty good ones. We're finally on all of the podcast aggregators including iTunes.

Grapevine also got a [forum](https://forum.grapevine.haus/) which uses your Grapevine login to sign into. If you're looking for a place for more long-form discussions around MUDs, this is the place.

I have also continued streaming over on Twitch every Monday at 12 PM EST doing ExVenture or Grapevine development. Join me over at [https://www.twitch.tv/smartlogictv](https://www.twitch.tv/smartlogictv).

# Next Month

Next month I would like to start working on more advanced features for the web client. I am starting with modals, and possibly adding in sound or text to speech next. I'm sure I'll keep working on Spigot as well, smoothing out that while it's still small. I would really like to start develing into ExVenture again. It's been too long since I worked on it.

[exventure]: https://exventure.org
[exventure-world]: https://exventure.world
[exventure-github]: https://github.com/oestrich/ex_venture
[grapevine]: https://grapevine.haus
[midmud]: https://midmud.com
[patreon]: https://www.patreon.com/exventure
[exventure-trello]: https://trello.com/b/PFGmFWmu/exventure
[grapevine-trello]: https://trello.com/b/bWZ00VpS/grapevine
[discord]: https://discord.gg/GPEa6dB
[mud-coders]: https://mudcoders.com/
