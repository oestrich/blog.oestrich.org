---
layout: post
categories:
- ruby
- sidekiq
title: LoggingWorker Gem
---

The last post was about logging workers/jobs as they happened in a manual process. From that I created a small gem that handles most of this for you. [LoggingWorker](https://github.com/smartlogic/logging_worker) is the gem. It's not up on rubygems yet, so you'll need to install it via `git` or `github` in bundler.

##### Gemfile
{% highlight ruby %}
gem 'logging_worker', :github => "smartlogic/logging_worker"
{% endhighlight %}

Once it's installed you'll need create the migrations.

{% highlight bash %}
$ rake logging_worker_engine:install:migrations
$ rake db:migrate db:test:prepare
{% endhighlight %}

To have workers start logging, you need to `prepend` the module in each worker. `prepend` is very important because the `#perform` method needs to be the `LoggingWorker::Worker#perform`.

Once this is done, everything is taken care of for you. You can continue to enhance your workers though. In one project I have it associate job runs to an object. To do this you need to set a custom `JobRun` class.

{% highlight ruby %}
class ImportantWorker
  include Sidekiq::Worker
  prepend LoggingWorker::Worker

  sidekiq_options({
    :job_run_class => ::JobRun
  })

  def perform(order_id)
    order = Order.find(order_id)
    job_run.order = order
  end
end
{% endhighlight %}

{% highlight ruby %}
class JobRun < LoggingWorker::JobRun
  belongs_to :order
end
{% endhighlight %}

You can also use the logger as the worker does its work. The log is saved inside the `JobRun` record.

{% highlight ruby %}
class ImportantWorker
  include Sidekiq::Worker
  prepend LoggingWorker::Worker

  def perform(order_id)
    order = Order.find(order_id)
    logger.info "Found the order, starting work..."
  end
end
{% endhighlight %}

If an error is raised it is captured, saved, and then reraised to let sidekiq perform it again.

Below is the full schema that comes with the `JobRun` class from the gem.

##### db/schema.rb
{% highlight ruby %}
create_table "job_runs", force: true do |t|
  t.string   "worker_class"
  t.string   "arguments",       array: true
  t.boolean  "successful"
  t.datetime "completed_at"
  t.text     "log"
  t.text     "error_class"
  t.text     "error_message"
  t.text     "error_backtrace", array: true
  t.datetime "created_at"
  t.datetime "updated_at"
end
{% endhighlight %}
