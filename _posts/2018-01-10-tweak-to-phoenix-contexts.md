---
layout: post
categories:
- elixir
- ex_venture
title: A Tweak to Phoenix Contexts
description: What ExVenture (a MUD engine) uses instead
date: 2018-01-10 9:00AM EST
---

I would like to start this post out by saying this isn't a negative post against the bounded context approach Phoenix is now geared towards. I wanted to showcase what has worked out really well with my side project, [ExVenture][exventure], which has grown to a fairly hefty size. As always you can see the source on [GitHub][exventure-github] and a running instance at [MidMUD][midmud].

# So what do you have instead?

Instead of the context approach where everything is hidden beneath the context I instead have several separate module namespaces that are my "contexts". Here is my `lib` folder structure:

```
lib/
├── data
├── ex_venture
├── game
├── metrics
├── networking
└── web
```

Each of those top level folders are a loose context.

## `Data.*`

This level contains all of my ecto schemas and modules that deal with data.

```
lib/data/
├── bug.ex
├── class.ex
├── config.ex
├── effect.ex
├── event.ex
└── ...
```

## `Game.*`

This level contains what I would call the business logic of the app. It has all of the user processing commands, gen servers, and is the core of the app.

```
lib/game/
├── command
├── command.ex
├── format
├── format.ex
├── item.ex
├── npc
├── npc.ex
├── room
├── room.ex
├── session
├── session.ex
└── ...
```

Inside this namespace I also split into more sub modules based in general on GenServer processes. For instance `Game.Session` has a lot of sub modules to help create a session process for when someone starts playing the app. This is fairly close to the general theme of the Bounded Contexts in Phoenix.

## `Metrics.*`

This level contains all of my [Prometheus][prometheus] metrics modules and is fairly slim.

## `Networking.*`

This level contains telnet related modules, including the ranch protocol.

## `Web.*`

This level contains the [Phoenix layer][add-phoenix-to-otp-app]. I should note that I consider ExVenture to be an OTP app first, and a Phoenix app a distant second.

```
lib/web
├── bug.ex
├── channels
├── class.ex
├── config.ex
├── controllers
├── endpoint.ex
├── filter.ex
├── gettext.ex
├── item.ex
├── npc.ex
├── pagination.ex
├── plug
├── prometheus_exporter.ex
├── room.ex
├── router.ex
├── supervisor.ex
├── templates
├── user.ex
├── views
└── ...
```

Small aside: I really like naming this module `Web` and not `ExVentureWeb` or similar, it really helps out my tab complete habit to be simply named `Web`.

# How This All Fits Together

So with all of these different modules as contexts, how does this all fit together? I will show this by giving an example of the NPC (the non-player characters in the game) slice.

## `Data.NPC`

This [module][data-npc-module] is a simple ecto schema, it simply validates the data in a changeset.

## `Game.NPC`

This is a [GenServer process][game-npc-module] for the NPC. It's a mix of an NPC struct and an NPC Spawner struct. I have a lot of sub modules for this dealing with events that the NPC may act on, respawning if it is killed, and other helper modules like the Supervisor for NPCs.

## `Web.NPC`

The [web module][web-npc-module] is a helper module to fit between the Phoenix layer and the core of my app. All of the controllers deal with this module and not a deeper part of the app. It handles creating NPCs, their spawners, adding items, and ensuring the live processes are kept up to date as all of these changes happen. This is probably the closest thing to a standard Phoenix context that I have.

# Conclusion

As you can see what I ended up with is only a slight tweak on the general principles behind Bounded Contexts.

I like having a general schema area and making sure nothing else goes in there. I like having a lot of modules that are related to the process they're attached to. I really like having the web layer barely know about the core of the app.

I hope this helps make you think about the structure of your app more and realize that bounded contexts are there to make you think about your app, and not be the only way.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[prometheus]: https://prometheus.io/
[add-phoenix-to-otp-app]: https://blog.oestrich.org/2017/09/adding-phoenix-to-otp-app/
[data-npc-module]: https://github.com/oestrich/ex_venture/blob/master/lib/data/npc.ex
[game-npc-module]: https://github.com/oestrich/ex_venture/blob/master/lib/game/npc.ex
[web-npc-module]: https://github.com/oestrich/ex_venture/blob/master/lib/web/npc.ex
