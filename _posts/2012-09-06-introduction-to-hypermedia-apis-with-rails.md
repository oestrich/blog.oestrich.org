---
layout: post
categories:
- rails
- api
- hypermedia
title: Introduction to Hypermedia APIs with Rails Presentation
---

Last week SmartLogic had an internal conference where I gave a presentation on hypermedia APIs with Rails, located [here](http://oestri.ch/presentations/intro-hypermedia). I created an example app that's on [github](https://github.com/oestrich/hypermedia_rails).  My favorite part about this talk was at the end when I attempted to change the routes the app serves up and see if my hal client still worked. It passed with flying colors.

I used the [frenetic](https://github.com/dlindahl/frenetic) gem to create my hal client. It's pretty verbose currently and let's just gloss over all those periods. It's a nice starting point for a HAL api.

{% highlight ruby %}
require 'frenetic'

MyAPI = Frenetic.new({
  'url'          => 'http://hypermedia.dev',
  'username'     => 'qxpRbQpqAw3YugKUpErW',
  'password'     => 'qxpRbQpqAw3YugKUpErW',
  'headers' => {
    'accept' => 'application/hal+json'
  }
})

class Order < Frenetic::Resource
  api_client { MyAPI }

  def self.orders
    api.get(api.description.links.orders.href).body.resources.orders.map do |order|
      new(order)
    end
  end
end

p Order.orders
{% endhighlight %}

#### Resources
* [presentation](http://oestri.ch/presentations/intro-hypermedia)
* [example app](https://github.com/oestrich/hypermedia_rails)
* [frenetic](https://github.com/dlindahl/frenetic)
