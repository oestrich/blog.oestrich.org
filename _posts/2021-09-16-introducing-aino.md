---
layout: post
category:
- elixir
title: Introducing the Aino framework
description: Aino is an experiment in writing a new HTTP framework
date: 2021-09-16 09:00 AM EDT
---

For the last few weeks I've been playing around with a new HTTP framework I'm calling Aino. [See it on GitHub](https://github.com/oestrich/aino).

Aino is built on top of [Elli][elli] and loosely based around [Ring][ring] from Clojure. It doesn't use Plug or Phoenix, though I'm sure some concepts were pulled from it given that's my day job!

## Why make a new framework? What are Aino's goals?

Aino is an experiment to try out a new way of writing HTTP applications on Elixir. It uses [elli][elli] instead of Cowboy like Phoenix and Plug. I wanted to see if there were different ways of writing web apps in Elixir. Is there a better way than MVC? Is there a better view layer? What would happen if we try a different HTTP library? Is there a better router? These are the sorts of questions I wanted to play around with.

With the current trajectory of Phoenix, I've been thinking a lot on how to keep "dead views" still alive. I personally think they have a seat at the table. I have listened to several mentors of mine from REST Fest talk about how we shouldn't keep state on the server for web/API clients. It's conversations and lessons like these that are guiding me towards exploring something like Aino.

Aino will likely stay as a very fast server side renderer and API framework. I have ideas on what to do for websockets if I ever need or want to add them in, elli has the capability through another library. These ideas come from a lot of experimenting in [Grapevine](https://github.com/oestrich/grapevine) and [Kalevala](https://github.com/oestrich/kalevala) with cowboy websockets directly, so that's my general plan if I end up there.

## Why the name Aino?

Aino is a character in the Kalevala. She's also the wife of Jean Sibelius, a composer that wrote works involving the Kalevala.

It's loosely themed around my other project [Kalevala](https://github.com/oestrich/kalevala), a text world builder. Plus my wife picked the name.

## What makes up Aino?

Aino is built around a handler. The handler receives a token for the request and then reduces over a list of middleware to generate a response. Tokens are simple maps with no defined structure, middleware are functions that take a single argument of the token.

Aino ships with a handful of common middleware, including parsing request POST body of a few MIME types (url encoded and JSON at the moment), header parsing, a tiny view wrapper around EEx, a _very_ simple router, and simple session stored in cookies.

The `Aino.Token` is taken from René Föhring's excellent [blog post](http://rrrene.org/2018/04/30/flow-elixir-token-approach-pros-and-cons/) and [ElixirConf 2018 talk](https://www.youtube.com/watch?v=ycpNi701aCs). I went with a simple map instead of a struct because this enables anyone to add any key in middleware without needing to extend a token struct. Using a simple map was also fits more in line with what [Ring][ring] uses for passing data between middleware.

## Example

If you want to see a more realistic example, I've written an RSS reader using Aino and it turned out well. [You can view the source here on GitHub](https://github.com/oestrich/aino-rss).

Below is a simple example of Aino using most of it's pieces to render "Hello, World".

```elixir
defmodule AinoExample.Application do
  @moduledoc false

  use Application

  @impl true
  def start(_type, _args) do
    children = [
      {Aino, callback: AinoExample.Handler, port: 3000}
    ]

    opts = [strategy: :one_for_one, name: AinoExample.Supervisor]
    Supervisor.start_link(children, opts)
  end
end

defmodule AinoExample.Handler do
  @behaviour Aino.Handler

  import Aino.Middleware.Routes, only: [get: 2]

  @impl true
  def handle(token) do
    routes = [
      get("/", &AinoExample.Pages.index/1)
    ]

    middleware = [
      Aino.Middleware.common(),
      &Aino.Middleware.Routes.routes(&1, routes),
      &Aino.Middleware.Routes.match_route/1,
      &Aino.Middleware.params/1,
      &Aino.Middleware.Routes.handle_route/1,
    ]

    Aino.Token.reduce(token, middleware)
  end
end

defmodule AinoExample.Pages do
  alias Aino.Token

  def index(token) do
    token
    |> Token.response_status(200)
    |> Token.response_header("Content-Type", "text/plain")
    |> Token.response_body("Hello, world!\n")
  end
end
```

Aino doesn't do a lot for you at the moment, and I'm not sure how much I want to abstract it away. I like the raw internals of HTTP right in your face. I have a few helper functions around redirecting and rendering HTML, so this will probably be the path forward for making things easier to use.

## Benchmarks

For initial benchmarks I was slightly blown away. Aino is only a few hundreds of lines of code, so being this fast makes sense. It's not doing a lot of work for you.

My benchmark script looks like this. It's running against the example above and a Phoenix application that was generated fresh, a single new route of "/hello" that points at the following controller, and generated via `MIX_ENV=prod mix release`.

I am including Phoenix as a point of comparison, I have not done Phoenix performance tweaking before so it might be able to get higher with more tweaks.

```elixir
defmodule HelloWeb.HelloController do
  use HelloWeb, :controller

  def index(conn, _params) do
    send_resp(conn, 200, "Hello, world!")
  end
end
```

```elixir
defmodule Bench do
  def test_aino() do
    Finch.build(:get, "http://localhost:3000/") |> Finch.request(Bench)
  end

  def test_phoenix() do
    Finch.build(:get, "http://localhost:4001/hello") |> Finch.request(Bench)
  end
end

{:ok, _pid} = Finch.start_link(name: Bench)
Benchee.run(%{
  "aino" => &Bench.test_aino/0,
  "phoenix" => &Bench.test_phoenix/0
}, time: 10)
```

```
❯ mix run bench.exs
Operating System: Linux
CPU Information: AMD Ryzen Threadripper 3970X 32-Core Processor
Number of Available Cores: 64
Available memory: 125.65 GB
Elixir 1.12.2
Erlang 24.0.5

Benchmark suite executing with the following configuration:
warmup: 2 s
time: 10 s
memory time: 0 ns
parallel: 1
inputs: none specified
Estimated total run time: 24 s

Benchmarking aino...
Benchmarking phoenix...

Name              ips        average  deviation         median         99th %
aino          12.21 K       81.88 μs    ±15.48%       79.82 μs      100.68 μs
phoenix        5.46 K      183.26 μs    ±10.87%      179.69 μs      297.52 μs

Comparison: 
aino          12.21 K
phoenix        5.46 K - 2.24x slower +101.38 μs
```

I'm very surprised at how well Aino / mostly Elli does in terms of rendering simple data. The results are encouraging enough that I've kept working on Aino.

## Current State

Aino is currently at an MVP stage, it works well enough to write simple applications. It's partially tested and mostly documented. There are a handful of pieces that still need better testing, and likely better documentation.

The last week or so I've mostly worked on documenting everything about Aino, and getting it set up to auto deploy documentation to a website before it's uploaded to hex. I now have [ainoweb.dev](https://ainoweb.dev) set up to build documentation on every git push so it'll always be up to date!

Please give the docs a look over and I'm more than happy to hear any feedback.

## Important Links

- [GitHub for Aino](https://github.com/oestrich/aino)
- [GitHub for Aino RSS](https://github.com/oestrich/aino-rss)
- [Documentation](https://ainoweb.dev/)

[elli]: https://github.com/elli-lib/elli
[ring]: https://github.com/ring-clojure/ring
