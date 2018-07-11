---
layout: post
categories:
- elixir
- otp
- ex_venture
title: Elixir OTP Supervision Tether
description: A supervision tree pattern for processes that may never recover
date: 2018-07-09 7:30PM EDT
---

I got to try out a new OTP Supervision tree pattern in [ExVenture][exventure], that I am going to call a tether.

#### Background

I started a new service, named [Gossip][gossip], that ExVenture creates a persistent websocket connection to.

When this service dies or is not alive when the ExVenture server starts, the websocket connection will crash or refuse to start entirely. It crashes immediately and then cannot reconnect. This ripples up the supervision tree taking the entire application down in short order.

To get around this, I added a layer of supervision trees for just that socket process. My tree looks like this when fully booted:

```
- ExVenture
  - Gossip.Supverisor
    - Gossip.Monitor
    - Gossip.Supverisor.Tether
      - Gossip.Socket
```

#### The Process

The tether supervision starts with no childspecs. After boot the Monitor (or the cluster leader) will start the Socket inside the tether. When the socket process dies, it will eventually crash the tether supervisor causing it to restart.

When the tether supervisor restarts, it will restart with no children breaking the restart loop ("cutting the tether".)

Eventually the monitor process will try to restart the Gossip socket which may crash, causing the tether to crash and so on. Either way the crashing process was contained and the application stayed up.

#### Conclusion

The code for this was pretty simple, you can see it on GitHub in the [gossip folder][exventure_gossip].

- [Gossip.Supervisor](https://github.com/oestrich/ex_venture/blob/dfe4394d285a853fff880391219161ed77f0eefc/lib/gossip/supervisor.ex)
- [Gossip.Supervisor.Tether](https://github.com/oestrich/ex_venture/blob/dfe4394d285a853fff880391219161ed77f0eefc/lib/gossip/supervisor.ex)
- [Gossip.Monitor](https://github.com/oestrich/ex_venture/blob/dfe4394d285a853fff880391219161ed77f0eefc/lib/gossip/monitor.ex)
- [Gossip.Socket](https://github.com/oestrich/ex_venture/blob/dfe4394d285a853fff880391219161ed77f0eefc/lib/gossip/socket.ex)

I learned of this at least part of this technique from Adam, over at the [MUD Coders Guild][mud-coders]. If you are interested in Elixir and multiplayer game programming, come check out the Slack and say Hi.

[exventure]: https://github.com/oestrich/ex_venture
[gossip]: https://gossip.haus/
[exventure_gossip]: https://github.com/oestrich/ex_venture/tree/master/lib/gossip
[mud-coders]: https://mudcoders.com/
