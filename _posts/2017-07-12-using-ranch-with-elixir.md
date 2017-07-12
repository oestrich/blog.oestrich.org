---
layout: post
categories:
- elixir
- erlang
- ranch
title: Using Ranch with Elixir
description: How to use the Erlang library, Ranch, with Elixir
date: 2017-07-12 3:00PM
---

I started playing around with [ranch][ranch] yesterday and didn't see any examples on how to use it with Elixir, or an example with a GenServer. I finally managed to get it going, here is how I did it.

## Start ranch

I have this module being started as a worker for my application. This will kick off the listener.

```elixir
defmodule App do
  def start_link() do
    :ranch.start_listener(make_ref(), :ranch_tcp, [{:port, 5555}], App.GenProtocol, [])
  end
end
```

The first parameter is a reference, I couldn't find any explanation on how it's used so I went with `make_ref`. The other thing that matters is the `App.GenProtocol` part, which is the callback module that ranch will use for starting new connections.

## Ranch Protocol

This part is fairly simple to get going as a non-GenServer process, but I really wanted to use GenServer since it handles a lot of extras for me. The tricky part was using `:proc_lib` to start the module and then entering the `:gen_server.enter_loop` after hooking up to the socket.

To note, the `transport` variable is `:ranch_tcp` which ends up calling into that erlang module.

```elixir
defmodule App.GenProtocol do
  use GenServer

  @behaviour :ranch_protocol

  def start_link(ref, socket, transport, _opts) do
    pid = :proc_lib.spawn_link(__MODULE__, :init, [ref, socket, transport])
    {:ok, pid}
  end

  def init(ref, socket, transport) do
    IO.puts "Starting protocol"

    :ok = :ranch.accept_ack(ref)
    :ok = transport.setopts(socket, [{:active, true}])
    :gen_server.enter_loop(__MODULE__, [], %{socket: socket, transport: transport})
  end

  def handle_info({:tcp, socket, data}, state = %{socket: socket, transport: transport}) do
    IO.inspect data
    transport.send(socket, data)
    {:noreply, state}
  end
  def handle_info({:tcp_closed, socket}, state = %{socket: socket, transport: transport}) do
    IO.puts "Closing"
    transport.close(socket)
    {:stop, :normal, state}
  end
end
```

This will create a simple echo server. You can connect via telnet with `telnet localhost 5555`. It is very easy to extend this to perform more complex actions.

## Example

##### Server
```bash
$ mix run --no-halt
Compiling 1 file (.ex)
Starting protocol
"Hi\r\n"
Closing
```

##### Client

```bash
$ telnet localhost 5555
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Hi
Hi
^]
telnet> Connection closed.
```

[ranch]: https://github.com/ninenines/ranch
