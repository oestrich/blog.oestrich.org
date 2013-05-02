---
layout: post
categories:
- rails
- api
- capybara
title: Booting Your Rails Server in a Script
---

A recent API Craft Google group post has gotten me playing around with hypermedia APIs again after having an extended stay in Android land. I updated my [hypermedia APIs](https://github.com/oestrich/hypermedia_rails) repo, specifically the [hypermedia.rb](https://github.com/oestrich/hypermedia_rails/blob/e8ddbacc9b8e9cee0da21739ccbd4345affa833b/hypermedia.rb) script.

I made it look similar to the [nerdword-api](https://github.com/smartlogic/nerdword-api) [script/client.rb](https://github.com/smartlogic/nerdword-api/blob/a28949c055ad0ba60136037f2063638c14039df8/script/client.rb) which boots a server in the script instead of relying on an external server beeing booted. This was taken from how Cucumber boots capybara.

Below is the code required:

    #!/usr/bin/env ruby

    require File.expand_path('../config/environment',  __FILE__)
    require 'capybara/server'

    Capybara.server do |app, port|
      require 'rack/handler/thin'
      Thin::Logging.silent = true
      Rack::Handler::Thin.run(app, :Port => port)
    end

    server = Capybara::Server.new(Rails.application, 8888)
    server.boot
