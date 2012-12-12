---
layout: post
categories:
- ruby
title: Rails Console for Your Gem
---

I have been creating a gem recently and will usually want to open up an `irb` session to play around with ideas. When opening the `irb` session I had to configure the gem and it started to get annoying. I got envious of `rails console` and decided to make my own `script/console` for the gem.

Below is what it ended up looking like.

##### script/console

    b = File.expand_path('../../lib/', __FILE__)
    $:.unshift lib unless $:.include?(lib)

    require 'my_gem'
    require 'irb'

    MyGem.configure do |config|
      config.name = "My Gem"
    end

    IRB.setup(nil)
    irb = IRB::Irb.new
    IRB.conf[:MAIN_CONTEXT] = irb.context
    irb.context.evaluate("require 'irb/completion'", 0)

    trap("SIGINT") do
      irb.signal_handle
    end
    catch(:IRB_EXIT) do
      irb.eval_input
    end

