---
layout: post
categories:
- rails
- sse
- python
title: Server Sent Events with Rails
description: Mini tutorial on doing Server Sent Events with Rails
---

I'm doing a fun side project and found a use case for Server Sent Events. I found out about these last [REST Fest][restfest] and have wanted to use them since then. This is a small tutorial on how I got it going.

##### events_controller.rb

``` ruby
class EventsController < ApplicationController
  include ActionController::Live

  def index
    response.headers['Content-Type'] = 'text/event-stream'

    sse = SSE.new(response.stream)

    begin
      redis = AppContainer.redis.call
      redis.subscribe("events:#{current_user.id}") do |on|
        on.message do |channel, event_id|
          event = Event.find_by(:id => event_id)

          sse.write(event.data, {
            :event => event.event_type,
            :id => event.id,
          }) if event
        end
      end
    rescue ClientDisconnected
      # This will only occur on a write
    ensure
      sse.close
    end

    render :nothing => true
  end
end
```

This is the entire controller that sends events. First you need to include `ActionController::Live` to include the `SSE` class and `ClientDisconnected` class.

I'm using redis pub/sub (also my first time trying this out.) I have a channel per user and publish event ids into the channel. Whenever a new message comes in I write a new event to the stream.

With this it will come out similar to this:

```
event: event-title
id: a6ffe373-8fcc-46de-abd1-e48767c7856b
data: {"hello":"world"}

```

Events have two new lines after them to distinguish them from one another.

To publish a new event:

``` ruby
redis.publish("event:#{user.id}", event.id)
```

## Drawbacks

I am using SSE outside of the browser and using a streaming http library to get load events. This works out well except for when the client disconnects. Rails only knows when the client disconnects after it tries writing to the stream.

My application currently doesn't generate enough events for this to be detected for a fairly long time. This means sockets won't get closed until a new event comes in. I would like to find a way around this as it will definitely die up resources.

[restfest]: http://restfest.org/
