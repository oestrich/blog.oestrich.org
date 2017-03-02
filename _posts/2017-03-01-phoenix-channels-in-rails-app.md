---
layout: post
categories:
- rails
- phoenix
- elixir
- ruby
title: Include Phoenix Channels in a Rails App
description: How to set up a small Phoenix app to bring channels into your Rails app
date: 2017-03-01 11:30PM
---

My side project [Worfcam][worfcam] has a home page that lists the most recent photo taken from a device. It used to poll every 3 seconds for updates to the photos. I now use Phoenix channels to push new photos to the home page. This is how I did that. I got part of the idea from [this blog post at akitonrails.com](http://www.akitaonrails.com/2015/12/09/ex-pusher-lite-part-1-phoenix-channels-and-rails-apps)

Security note: it should be pointed out that this has very lax security as it trusts the rails process and elixir process are on the same machine and firewalled away from the world outside of connecting through nginx.

## Rails Configuration

First include `phoenix.js` in your javascript. I managed to get it included by vendoring a hand compiled version straight from the [phoenix github][phoenix-github]. After this is in place, you write your javascript the same way as you would a normal Phoenix app.

To connect the proper socket I load it from the environment. I add a new method to my `AppContainer` and use it in the javascript when connecting to the Phoenix socket. For local development this is `ws://localhost:5300/socket`. On production it is `wss://example.com/socket`.

```ruby
class AppContainer
  # ...
  let(:channels_url) do
    ENV["CHANNELS_URL"]
  end
end
```

## Pushing Events

The phoenix app will have the following route:

```elixir
  scope "/", Channels do
    resources "/events", EventController, only: [:create]
  end
```

The controller will simply broadcast whatever gets sent in. This is why it trusts the network is locked down. A simple way to add authentication to this could be basic auth or digest auth. Or even OAuth 2.0 server to server workflow.

```elixir
defmodule Channels.EventController do
  use Channels.Web, :controller

  def create(conn, %{"topic" => topic, "event" => event, "body" => body}) do
    Channels.Endpoint.broadcast(topic, event, body)
    conn |> send_resp(204, "")
  end
end
```

I have the Ecto Repo pointing to the same database and the models have a schema that matches what rails uses. My channels enforce that only the correct users can connect to channels. Channels will be straight forward normal channels.

To push events over from Rails I have a sidekiq job that sends the data required by the create action. Once in place events flow from Rails to Phoenix to the web browser.

```ruby
class EventPushWorker
  include Sidekiq::Worker

  def perform(topic, event, body)
    return if Rails.env.test?
    response = Faraday.post(AppContainer.channels_event_url) do |req|
      req.headers["Content-Type"] = "application/json"
      req.body = {
        topic: topic,
        event: event,
        body: body,
      }.to_json
    end
  end
end
```

## nginx Configuration

The only tricky part from here is getting Rails to talk to Phoenix in production. Local development works because you can talk directly to the server. For production I have nginx set up a location before the default location that passes to Rails.

```nginx
upstram worfcam {
  server localhost:5000;
}

upstream channels {
	server localhost:5300;
}

location /socket {
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection $connection_upgrade;
	proxy_set_header Origin $http_origin;
	proxy_pass http://channels;
}

location / {
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto https;
	proxy_set_header Host $http_host;
	proxy_redirect off;
	proxy_pass http://worfcam;
}
```

Once this is in place, your Rails app doesn't even need to know that it's talking to Phoenix from the front end.

[worfcam]: https://worfcam.com
[phoenix-github]: https://github.com/phoenixframework/phoenix
