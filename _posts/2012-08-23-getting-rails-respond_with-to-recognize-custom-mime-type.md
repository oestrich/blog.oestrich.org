---
layout: post
categories:
- rails
- api
title: Getting Rails respond_with to recognize custom mime types
---

# [{{ page.title }}]({{ page.url }})
<span>Posted on {{ page.date | date_to_string }}</span>

I am writing a sample hypermedia API in Rails, [here](https://github.com/oestrich/hypermedia_rails), and recently came across some trouble trying to get Rails to recognize "application/hal+json" as a JSON format.

I ended up having to override respond_with to make it understand the HAL format. Not exactly what I hoped for, but it works.

##### config/initializers/mime_types.rb
    Mime::Type.register "application/hal+json", :hal

    ActionDispatch::ParamsParser::DEFAULT_PARSERS[Mime::Type.lookup('application/hal+json')] = 
      lambda do |body|
        JSON.parse(body)
      end

##### app/controllers/application_controller.rb<
    class ApplicationController < ActionController::Base
      ...

      private
      def respond_with(resource, options = {})
        super(resource, options) do |format|
          format.hal do
            render options.merge(:json => resource)
          end
        end
      end
    end

##### app/controllers/home_controller.rb
    HomeController < ApplicationController
      respond_to :hal

      ...
    end

This method doesn't give the benefit of auto status codes based on the HTTP verb. I did however get a monkey patch to work:

##### config/initializers/monkey_patching.rb
    module ActionController
      class Responder
        def to_hal
          @format = :json
          api_behavior(nil)
        end
      end
    end

The overload of respond_with is now no longer necessary, but we're accessing a protected method.

Resources:

1.  [http://stackoverflow.com/questions/8700332/rails-3-and-json-default-renderer-but-custom-mime-type](http://stackoverflow.com/questions/8700332/rails-3-and-json-default-renderer-but-custom-mime-type)
1.  [http://http://api.rubyonrails.org/classes/Mime/Type.html](http://api.rubyonrails.org/classes/Mime/Type.html)
1.  [https://github.com/oestrich/hypermedia_rails/commit/1fc2d721aa80c384867d7d374baf3eada6e166e6](https://github.com/oestrich/hypermedia_rails/commit/1fc2d721aa80c384867d7d374baf3eada6e166e6)
1.  [http://api.rubyonrails.org/classes/ActionController/Responder.html](http://api.rubyonrails.org/classes/ActionController/Responder.html)
