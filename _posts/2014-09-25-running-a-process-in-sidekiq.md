---
layout: post
categories:
- ruby
- sidekiq
- celluloid
title: Running an extra process inside of sidekiq
---

For one of my side projects I wanted to have an always on process. I didn't really want to have to set up something like daemon-kit for it as that felt like too much for what this was going to do. I remembered that [sidetiq][sidetiq] was able to accomplish this and started looking at the source code.

Sidetiq very simply starts running inside the [sidekiq][sidekiq] process by using [celluloid][celluloid]. This was the first time I played with celluloid so it was a good time to start learning about it.

Launching your process inside of sidekiq is easy.

{% highlight ruby %}
if Sidekiq.server?
  MySupervisor.run!
end
{% endhighlight %}

A supervisor simply keeps celluloid actors going if they die. See [supervisors][celluloid-supervisors] for more information. My supervisor looked like:

{% highlight ruby %}
class MySupervisor < Celluloid::SupervisionGroup
  supervise CelluloidActor, :as => :actor
  supervise OtherCelluloidActor, :as => :other_actor
  supervise Orchestrator, :as => :orchestrator
end
{% endhighlight %}

I created an orchestrator actor that does all of the main work for me. It looks at redis and pops messages off a queue as they are entered.

{% highlight ruby %}
class RemoteOrchestrator
  include Celluloid
  include Celluloid::Logger

  def initialize(redis_pool = AppContainer.redis_connection_pool)
    @redis_pool = redis_pool

    info "Starting orchestrator for remote connections"

    after(1) do
      loop!
    end
  end

  def loop!
    message = connection_pool.with do |redis|
      redis.brpop("requests", :timeout => 5)
    end

    unless message
      async.loop!
      return
    end

    message = JSON.parse(message.last)

    info "Received: #{message.inspect}"

    worker = workers[message["client"]]
    worker.async.process(message["body"])

    async.loop!
  end

  private

  def workers
    @workers ||= {
      "actor" => Celluloid::Actor[:actor],
      "other_actor" => Celluloid::Actor[:other_actor],
    }
  end

  def connection_pool
    @redis_pool
  end
end
{% endhighlight %}

The important part about this class is the `after(1)` call. It queues the block to run in 1 second, which puts it in the background. The actor also loops itself by sending another `loop!` message async at the end of `loop!`. This makes sure it will continuing running.

I've run this in my side project for a few months now and I haven't noticed any ill effects.

[sidekiq]: http://sidekiq.org/
[sidetiq]: https://github.com/tobiassvn/sidetiq
[celluloid]: https://github.com/celluloid/celluloid
[celluloid-supervisors]: https://github.com/celluloid/celluloid/wiki/Supervisors
