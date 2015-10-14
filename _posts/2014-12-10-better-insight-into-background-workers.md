---
layout: post
categories:
- ruby
- sidekiq
title: Better Insight Into Background Workers
---

One of the projects I'm working on now does thousands of background jobs a day. It uses sidekiq and handles errors fairly well by letting them retry normally. This works great in a staging environment as you know more or less what is happening all the time.

Once we got this into production we quickly found out we needed better insight into what the system was doing all the time. For this I came up with idea to log each job as it's happening, fairly automatically.

When a job starts it immediately creates a `JobRun` which logs the start time, worker class, and the arguments passed in. It has an ensure block to always make sure it saves the finish time of a job, even if it wasn't successful.

Below is the general layout of our worker classes.

```ruby
class MyWorker < AppWorker
  include Sidekiq::Worker

  def perform(arg1, arg2)
    start_perform!(arg1, arg2)

    # do work

    successful!
  ensure
    finish_perform!
  end
end
```

The app has an admin panel that lets us easily view each job run as it's happening. This way you can see and have a record of what the system did and how many times a job failed.

A few additions in the future I want to look into is having each job have a log. So as the worker is doing it's job it will save into a text column that we can look at after it's done to see exactly what happened. It would be good to be able to tie a `JobRun` between retries as well. So if a job failed we could link to the next run of that exact job.
