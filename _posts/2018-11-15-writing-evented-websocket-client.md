---
layout: post
categories:
- elixir
- gossip
title: Writing an Evented WebSocket Client
description: How I wrote an evented websocket client for Gossip in Elixir
date: 2018-11-15 8:30 EDT
---

I recently did a pretty big refactor for the [Gossip Elixir][gossip-elixir] client. I'd like to show off what I did for that. For a brief background, [Gossip][gossip] sends events over the websocket connection. A sample event might be:

```json
{
  "event": "channels/broadcast",
  "ref": "89036074-446f-41ab-b87a-44ef1f962f2e",
  "payload": {
    "channel": "gossip",
    "message": "Hello everyone!",
    "game": "ExVenture",
    "name": "Player"
  }
}
```

You can see all of the events over at the [Gossip Docs](https://gossip.haus/docs).

## Client Flow

The client connects to Gossip via the [`Gossip.Socket`](https://github.com/oestrich/gossip-elixir/blob/master/lib/gossip/socket.ex) server. This is a Websockex process, which is a little different than your standard GenServer. It might eventually be swapped out to be a [Gun](https://github.com/ninenines/gun) client.

The flow of data goes as such:

#### New websocket frame

[Full code](https://github.com/oestrich/gossip-elixir/blob/0c22c6ee835760dba429ba958c89124dd484ef38/lib/gossip/socket.ex#L45)

```elixir
defmodule Gossip.Socket do
  # ...

  def handle_frame({:text, message}, state) do
    case Events.receive(state, message) do
      {:ok, state} ->
        {:ok, state}

      {:reply, message, state} ->
        {:reply, {:text, Poison.encode!(message)}, state}

      # other return values
    end
  end

  # ...
end
```

When the client gets a new websocket frame, the socket process calls down to a lower level module.

#### Handling the event

[Full code](https://github.com/oestrich/gossip-elixir/blob/0c22c6ee835760dba429ba958c89124dd484ef38/lib/gossip/socket/events.ex#L16)

```elixir
defmodule Gossip.Socket.Events do
  # ...

  def receive(state, message) do
    with {:ok, message} <- Poison.decode(message),
         {:ok, state} <- process(state, message) do
      {:ok, state}
    else
      {:reply, message, state} ->
        {:reply, message, state}
    end
  end

  # ...
end
```

The frame is decoded to an event and then processed.

#### Pattern matching on the event

[Full code](https://github.com/oestrich/gossip-elixir/blob/0c22c6ee835760dba429ba958c89124dd484ef38/lib/gossip/socket/events.ex#L45)

```elixir
defmodule Gossip.Socket.Events do
  # ...

  def process(state, message = %{"event" => "channels/" <> _}) do
    Core.handle_receive(state, message)
  end

  def process(state, message = %{"event" => "players/" <> _}) do
    Players.handle_receive(state, message)
  end

  # ...
end
```

Each event has a general scheme of `noun/verb`, such as `channels/broadcast`. This pattern matches on the just the noun part and pushes it lower into the module for processing.

#### Pattern matched on the specific event

[Full code](https://github.com/oestrich/gossip-elixir/blob/0c22c6ee835760dba429ba958c89124dd484ef38/lib/gossip/socket/core.ex#L92)

[Full code](https://github.com/oestrich/gossip-elixir/blob/0c22c6ee835760dba429ba958c89124dd484ef38/lib/gossip/socket/core.ex#L126)

```elixir
defmodule Gossip.Socket.Core do
  # ...

  def handle_receive(state, message = %{"event" => "channels/broadcast"}) do
    process_channel_broadcast(state, message)
  end

  def process_channel_broadcast(state, %{"payload" => payload}) do
    message = %Message{
      channel: payload["channel"],
      game: payload["game"],
      name: payload["name"],
      message: payload["message"],
    }

    core_module(state).message_broadcast(message)

    {:ok, state}
  end

  # ...
end
```

Here the event is fully pattern matched and calls to the internal function. I like to do it this way to keep it similar to GenServers and keeping the `handle_*` functions skinny.

## Testing

With the client broken up into fairly small pieces, it helps encourage testing and keeps it simple to test.

Prior to this refactor, the Gossip client had almost no tests. This is due to each event module being broken up into separate elixir modules and having separate processing functions for each event.

[See testing in action.](https://github.com/oestrich/gossip-elixir/blob/0c22c6ee835760dba429ba958c89124dd484ef38/test/gossip/socket/core_test.exs#L109)

## Conclusion

I hope you poke around the rest of the client. I also recommend taking a look around the server side of Gossip since that got a big refactor as well. One really cool section is the ["event router" macro](https://github.com/oestrich/gossip/blob/5eb4d4a622699f2182e7f75711dfce5f67118937/lib/web/socket/router.ex#L20-L42) I set up to generate the server side receive functions.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[gossip]: https://gossip.haus
[gossip-elixir]: https://github.com/oestrich/gossip-elixir
