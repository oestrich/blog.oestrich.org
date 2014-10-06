---
layout: post
categories:
- ruby
- farday
- redis
title: "\"Remote\" Faraday: Respecting API Rate Limits"
---

This post will continue with the last post about [Running an extra process inside of sidekiq]({% post_url 2014-09-25-running-a-process-in-sidekiq %}).

My side project connects to a lot of APIs ([themoviedb.org][tmdb], [thetvdb.com][tvdb], [GoodReads][goodreads], and [BoardGameGeek][bgg]). Each of one of these has their own rate limit. Previously I was scheduling sidekiq jobs 10 seconds apart to make sure I didn't hit any of their limits. This was really lame and I wanted a nicer way of handling the rate limit.

What I came up with was what I referred to as "remote" faraday. The faraday connection is in a single spot inside of the extra process I stuck inside of sidekiq.

I have classes that handle each API for two specific needs, searching for items and viewing a single item. A sample for BoardGameGeek is below.

{% highlight ruby %}
class BGG
  def search(board_game_name)
    response = remote_faraday.
      get(:bgg, "/xmlapi/search?search=#{CGI.escape(board_game_name)}")
    xml = Nokogiri::XML(response)
    # process the xml
  end

  def board_game(board_game_id)
    request = remote_faraday.
      get(:bgg, "/xmlapi/boardgame/#{board_game_id}")
    xml = Nokogiri::XML(request)
    # process the xml
  end

  private

  def remote_faraday
    @remote_faraday ||= RemoteFaraday.new
  end
end
{% endhighlight %}

There's not much difference here from a regular faraday connection and my "remote" connection. This was nice because this class shouldn't care about the change.

Here is what the RemoteFaraday class looks like:

{% highlight ruby %}
class RemoteFaraday
  MAX_LOOPS = 200 # a minute or so of waiting

  class TimeoutError < StandardError
  end

  def initialize(container = AppContainer)
    @redis_pool = container.redis_connection_pool
    @uuid_generator = container.uuid_generator
  end

  def get(client, path)
    uuid = @uuid_generator.uuid

    @redis_pool.with do |redis|
      redis.lpush("remote-faraday:requests", {
        :method => :get,
        :client => client,
        :path => path,
        :uuid => uuid,
      }.to_json)
    end

    count = 0
    begin
      response = @redis_pool.with do |redis|
        redis.get(redis_key(client, :get, path, uuid))
      end
      sleep 0.3
      count += 1
    end while response.nil? && count < MAX_LOOPS

    if response.nil?
      raise TimeoutError
    end

    @redis_pool.with do |redis|
      redis.del(redis_key(client, :get, path, uuid))
    end

    response
  end

  private

  def redis_key(client, method, path, uuid)
    "#{client}:response:#{method}:#{path}:#{uuid}"
  end
end
{% endhighlight %}

The class takes a redis pool and UUID generator from an application container. The UUID is used to make sure each request is saved uniquely inside of redis. We don't want to overwrite a different request/response.

The RemoteFaraday class pushes a request into the redis queue and waits about a minute for a response. In order to not fill up redis we clear out the key that the response was saved into.

This class only handles `GET`s at the moment because I haven't needed to use any other method yet.

The other end of the queue is the `RemoteOrchestrator` that was seen in the previous post. Here is the full one.

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
    message = @redis_pool.with do |redis|
      redis.brpop("remote-faraday:requests", :timeout => 5)
    end

    unless message
      async.loop!
      return
    end

    message = JSON.parse(message.last)

    info "Received: #{message.inspect}"

    worker = workers[message["client"]]
    worker.async.send(message["method"], message["path"], message["uuid"])

    async.loop!
  end

  private

  def workers
    @workers ||= {
      "bgg" => Celluloid::Actor[:bgg],
    }
  end

  def connection_pool
    @redis_pool
  end
end
{% endhighlight %}

This is virtually the same as what I put in the last post. The `#loop!` method calls itself after each message or breaking from a timeouted `brpop`.

Below is the `RemoteConnection` class that uses faraday to connect to an API.

{% highlight ruby %}
class RemoteConnection
  include Celluloid
  include Celluloid::Logger

  def initialize(connection_pool = AppContainer.redis_connection_pool)
    @connection_pool = connection_pool
  end

  def get(path, uuid)
    exclusive do
      info "Fetching #{path} with uuid #{uuid}"
      response = connection.get(path)

      @connection_pool.with do |redis|
        redis.set(redis_key(:get, path, uuid), response.body)
        info "Set response to #{redis_key(:get, path, uuid)}"
      end

      sleep timeout
    end
  end

  def timeout
    1
  end

  private

  def redis_key(method, path, uuid)
    "#{client}:response:#{method}:#{path}:#{uuid}"
  end

  def connection
    @connection ||= Faraday.new(host)
  end
end
{% endhighlight %}

The important bit in the previous class is `exclusive`. This is a celluloid directive that makes sure only 1 worker ever runs what is inside of the block. Since the entire method is inside the block only 1 connection will happen.

Below is a specific subclass for BoardGameGeek.

{% highlight ruby %}
class RemoteBGG < RemoteConnection
  def host
    "http://boardgamegeek.com"
  end

  def client
    :bgg
  end
end
{% endhighlight %}

This was a fun little project that has let me speed up my sidekiq jobs since I no longer needed to arbitrarily wait 10 seconds in between jobs. I've thought about making this into a gem, but I haven't had a reason to yet. I'm also not sure how useful it would be as a gem.

[tmdb]: http://www.themoviedb.org/
[tvdb]: http://thetvdb.com/
[goodreads]: http://www.goodreads.com/
[bgg]: http://boardgamegeek.com/
