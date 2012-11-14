---
layout: post
categories:
- rails
- ruby
title: ActiveSupport::Notifications for Metrics
---

The other week I had to speed up an end point of an API I'm working on, and since I wanted to do it right I used metrics. A [coworker](https://twitter.com/nontrivialzeros) pointed me on to ActiveSupport::Notification.instrument previously so I gave it a go here.

I ended up creating a module that you can include into a class which will instrument every method. I don't think it's the nicest code ever but it was handy in getting a somewhat large class set up quickly.

It's available in this [gist](https://gist.github.com/4072523), but I've also included it below.

##### lib/notifications.rb
    module Notifications
      extend ActiveSupport::Concern

      module ClassMethods
        def method_added(method_name)
          @methods ||= []
          return if @methods.include?(method_name) || method_name =~ /_old$/
          @methods << method_name

          class_eval %{alias #{method_name}_old #{method_name}}

          define_method(method_name) do |*args|
            instrument_name = "#{method_name}.#{self.class.name.underscore}"
            ActiveSupport::Notifications.instrument(instrument_name) do
              send("#{method_name}_old", *args)
            end
          end
        end
      end
    end

You also need to set up something that will log the instruments so you can see them.

##### config/initializers/notifications.rb
    ActiveSupport::Notifications.subscribe(/my_class$/) do |*args|
      event = ActiveSupport::Notifications::Event.new(*args)

      Rails.logger.warn "%7.2fms %s" % [event.duration, event.name]
    end

### Resources
1. [ActiveSupport::Notifications](http://api.rubyonrails.org/classes/ActiveSupport/Notifications.html)
