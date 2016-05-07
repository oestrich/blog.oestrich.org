---
layout: post
categories:
- rails
- sidekiq
- redis
title: Sidekiq Job Locking - Lock Around an ID
description: Limit Sidekiq jobs to a single instance per ID.
date: 2016-05-07 12:00 PM
---

For one of my projects at SmartLogic, we process events in the background from webhooks. These talk to an API that need to be rate limited. Because of this we made a [Sidekiq][sidekiq] job that locks itself so only one job will run per ID.

For a better understanding of what this will accomplish, here is a brief example:

```ruby
MyWorker.perform_async(1) # will process
MyWorker.perform_async(2) # will process
MyWorker.perform_async(1) # will immediately exit, 1 still processing

sleep 10 # all jobs have finished

MyWorker.perform_async(1) # will process
```

### Recurrence

It should be pointed out that the worker example will tick with [Sidetiq][sidetiq]. This ensures we have a stready stream of jobs into the queue. The worker example will also reschedule itself to beat the tick if there are events to process.

```ruby
class MySchedulerWorker
  include Sidekiq::Worker
  include Sidetiq::Schedulable

  recurrence { hourly.minute_of_hour(0, 15, 30, 45) }

  def perform
    User.all.each do |user|
      MyWorker.perform_async(user.id)
    end
  end
end
```

### Worker

Let's start with what it looks like to use the locked job. This job will process an event stream of a user and pull off a certain amount of events each job run. The job will lock around the user, not the class itself.

```ruby
class MyWorker
  include Sidekiq::Worker
  include LockedJob

  def perform(user_id)
    return if already_processing?(shop_id)

    lock_processing!(user_id)

    # ... process events

    if events_left?
      @schedule_next_run = true
    end
  ensure
    clear_processing_lock(user_id)

    if @schedule_next_run
      self.class.perform_async(user_id)
    end
  end

  private

  # ...

  def processing_key(user_id)
    "users:#{user_id}:processing:#{jid}"
  end

  def processing_key_star(user_id)
    "users:#{user_id}:processing:*"
  end
end
```

To start out we check for `#already_processing?`. If something is already processing, we just want this job to stop now. Once we know it's safe to start working, `#lock_processing!`.

This job will reschedule itself if there are still items to process, so we set an instance variable to trigger a new run once the lock has been cleared.

The last two private methods are very important for this. They have a unique key for redis that uses the job id from sidekiq and then a star version that we'll use to determine how many of this job is currently running.

### LockedJob

Note: AppContainer is a container for configuration in my projects. `#redis_pool` is a [connection_pool][connection_pool] for redis.

```ruby
module LockedJob
  SIXTY_MINUTES = 60 * 60

  private

  def already_processing?(model_id)
    AppContainer.redis_pool.with do |redis|
      redis.keys(processing_key_star(model_id)).count >= 1
    end
  end

  def lock_processing!(model_id)
    AppContainer.redis_pool.with do |redis|
      redis.setex(processing_key(model_id), SIXTY_MINUTES, "true")
    end
  end

  def clear_processing_lock!(model_id)
    # We are done processing, clean out the key
    AppContainer.redis_pool.with do |redis|
      redis.del(processing_key(model_id))
    end
  end
end
```

This module has the special methods that the worker used. `#already_processing?` checks with the star key to see how many other jobs are running. This can easily be set to larger numbers if you want to lock at different rates.

`#lock_processing!` sets a key that will expire in 60 minutes. We set an expiration to make sure that other jobs can still start if this one hangs. We don't want to lock forever.

`#clear_processing_lock!` deletes the model specific key to allow other jobs to start.

## Benefits of Locking

This set up has worked very successfully for the last few months. We went from a separate job per event and trying to rate limit around that. It was very complicated to limit when fanned out like that. The worst version of that was very similar to this with keys for locking, but cycled hundreds of millions of jobs. A very large strain on the system for doing virtually nothing.

It helps out a lot with handling rate limits for talking to external APIs.

[sidekiq]: https://github.com/mperham/sidekiq
[sidetiq]: https://github.com/tobiassvn/sidetiq
[connection_pool]: https://github.com/mperham/connection_pool
