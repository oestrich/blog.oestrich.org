---
layout: post
categories:
- elixir
- otp
- cowboy
- gossip
title: Elixir Cowboy Websocket Handler
description: How to set up a Cowboy websocket handler in Phoenix
date: 2018-08-10 9:30AM EDT
---

For my side project [Gossip][gossip] I wanted to have a websocket connection for non-Phoenix connections. I did this by going straight to Cowboy and using a handler at that level. This explains how I did this for Gossip.

Gossip is a cross game chat service for MUDs, [check it out][gossip].

## The Handler

The full websocket handler is [here on GitHub][websocket-github].

```elixir
defmodule Web.SocketHandler do
  @behaviour :cowboy_websocket_handler

  def init(_, _req, _opts) do
    {:upgrade, :protocol, :cowboy_websocket}
  end

  def websocket_init(_type, req, _opts) do
    Logger.info("Socket starting")
    {:ok, req, %State{status: "inactive"}}
  end

  def websocket_handle({:text, message}, req, state) do
    with {:ok, message} <- Poison.decode(message),
         {:ok, response, state} <- Implementation.receive(state, message) do
      {:reply, {:text, Poison.encode!(response)}, req, state}
    else
      {:ok, state} ->
        {:ok, req, state}

      _ ->
        {:reply, {:text, Poison.encode!(%{status: "unknown"})}, req, state}
    end
  end
end
```

This is a snipped version of the full file, but the basics are here. This shows the websocket upgrading, a cowboy and websockets requirement. The `init` function upgrades to websockets and the websocket_init function is called after the upgrade.

The other function shown is when a new message is received. The message is JSON (or should be), so it gets parsed and then run through an implementation module elsewhere. Depending on the response from that submodule, different responses will get send back.

There are a few other cool things in the real module, so I encourage you to [check it out][websocket-github].

## Ping/Pong

One thing I had to add that I thought was included as part of the cowboy handler was a pong response to a client side ping. This actually crashed the websocket process a few times so I needed to add this as per the websocket spec:

```elixir
def websocket_handle({:ping, message}, req, state) do
  {:reply, {:pong, message}, req, state}
end
```

## Phoenix Configuration

Since we're using a lower level websocket (than Phoenix) we have to manually set up the cowboy dispatcher. This configuration shows the cowboy websocket handler along with a separate Phoenix channel, since I want to have both options.

If you go this route, you need to manually specify any Phoenix channels from here on out.

```elixir
config :gossip, Web.Endpoint,
  http: [dispatch: [
    {:_, [
      {"/socket", Web.SocketHandler, []},
      {"/chat/websocket", Phoenix.Endpoint.CowboyWebSocket, {Phoenix.Transports.WebSocket, {Web.Endpoint, Web.UserSocket, :websocket}}},
      {:_, Plug.Adapters.Cowboy.Handler, {Web.Endpoint, []}}
    ]}
  ]]
```

## Conclusion

Setting up your own lower level websocket in Elixir/Phoenix turned out to be pretty simple. I am happy I went this route so I didn't have to worry about forcing the higher level Phoenix channels protocol on top of external clients.

If you're curious about more of the events [Gossip][gossip] sends, the [docs are available here][gossip-docs].

[gossip]: https://gossip.haus/
[gossip-docs]: https://gossip.haus/docs
[websocket-github]: https://github.com/oestrich/gossip/blob/master/lib/web/socket_handler.ex
