---
layout: post
categories:
- ruby
title: Creating a Faraday middleware
---

I've been playing around with [Faraday](https://github.com/technoweenie/faraday) recently and wanted to make a middleware. The guide on the wiki leave a little to be desired. Googling didn't seem to help much either. I eventually found a [Faraday Middleware](https://github.com/pengwynn/faraday_middleware) github repo that helped me.

Here is a sample middleware that I ended up using:

    class MyMiddleware < Faraday::Middleware
      def call(env)
        env[:request_headers]["My-Custom-Header"] = "my-custom-value"

        @app.call(env)
      end
    end

    connection = Faraday.new("http://google.com/") do |faraday|
      faraday.use MyMiddleware
      faraday.adapter Faraday.default_adapter
    end
