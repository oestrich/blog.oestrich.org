---
layout: post
categories:
- ruby
- webmachine
title: Trying out WebMachine
---

Yesterday I tried out [webmachine-ruby](https://github.com/seancribbs/webmachine-ruby). Overall it left a good impression on me. One thing I really liked was the separation of a single resource and a collection resource.

    App = Webmachine::Application.new do |app|
      app.routes do
        add ["orders"], OrdersResource
        add ["orders", :id], OrderResource
      end
    end

Any call to "/orders" will go to the OrdersResource, since you're acting on a collection, and any call with an "id" in it will go to the single resource.

I also really liked the idea of callbacks on a resource. To point out one, in order to get a resource to respond to a certain method, you add it to the `#allowed_methods`:

    class OrderResource 
      def allowed_methods
        ["GET", "POST"]
      end

      def content_types_provided
        [["application/json", :to_json]]
      end
    end

The `#content_types_provided` callback will call `#to_json` if it sees "application/json".

One pretty big issue I faced was a lack of examples on how to use its statemachine. The example apps listed only do `GET` and `POST` requests but there wasn't an example on how do `DELETE` or `PUT` something. Luckily I had access to [someone who has used it before](https://github.com/samwgoldman) and could help me with the issues I came across. I'm hoping this example app can be of use to someone else looking into WebMachine.

My resulting example app is [here](https://gist.github.com/3638605) as a gist and I've embedded the main app file.

<script src="https://gist.github.com/3638605.js?file=app.rb"></script>
