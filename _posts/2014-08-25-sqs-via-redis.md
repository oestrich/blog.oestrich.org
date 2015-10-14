---
layout: post
categories:
- ruby
- sqs
- aws
- redis
title: Local SQS development with Redis
---

For [SmartChat](http://github.com/smartlogic/smartchat-api) we used [SQS](http://aws.amazon.com/sqs/). We used this as a way of sending messages for background workers instead of Sidekiq or Resque. This worked well, but for local development having to use AWS resources wasn't ideal. We set about creating a way of using Redis instead.

The SQS interface that we used was very simple. It has two methods that were used, `#send_messsage` and `#poll`. With this we can make a redis version.

##### redis_queue.rb
```ruby
require 'redis'

RedisMessage = Struct.new(:body)

class RedisQueue
  def initialize(redis)
    @redis = redis
  end

  def poll(&block)
    loop do
      queue, message = @redis.brpop("smartchat-queue")
      # queue could be nil if a timeout happened
      if queue
        block.call(RedisMessage.new(message))
      end
    end
  end

  def send_message(message_json)
    @redis.lpush("smartchat-queue", message_json)
  end
end
```

To use this, we have a `AppContainer` switch to it for development purposes and use it normally.

##### config/initializers/app_container.rb
```ruby
#...
let(:queue) do
  if Rails.env.development? || Rails.env.all?
    require 'redis_queue'
    RedisQueue.new(redis)
  else
    AWS::SQS.new.queues.named(sqs_queue_name)
  end
end
#...
```

##### libexec/smartchat-daemon.rb
```ruby
AppContainer.queue.poll do |msg|
  body = JSON.parse(msg.body)

  # do work
end
```

## Related files
- [redis_queue.rb](https://github.com/smartlogic/smartchat-api/blob/master/worker/lib/redis_queue.rb)
- [smartchat-daemon.rb](https://github.com/smartlogic/smartchat-api/blob/master/worker/libexec/smartchat-daemon.rb)
- [container.rb](https://github.com/smartlogic/smartchat-api/blob/master/config/initializers/container.rb)
