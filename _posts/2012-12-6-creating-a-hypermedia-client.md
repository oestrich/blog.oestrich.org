---
layout: post
categories:
- ruby
- rails
- hypermedia
title: Creating a Hypermedia Client
---

Over the past few weeks I wrote a client for a hypermedia api that I created. I think I ended up coming across a pattern that I like in regards to how it's structured.

I have three basic object types: services, loaders, and resources.

The http client used is the gem [Farday](http://github.com/technoweenie/faraday).

#### Services
A service is an object that performs an action. For example I have one service that registers users.

```ruby
class UserRegistrationService < Struct.new(:email, :password)
  def perform
    client.post(user_registration_url, {
      :user => {
        :email => email,
        :password => password
      }
    }
  end
end
```

#### Resources
A resource is a data only object. For example I might have a resource for an order.

```ruby
class Order
  def self.load(json_string)
    new(JSON.parse(json_string))
  end

  def initialize(attrs)
    @email = attrs.fetch("email")
    @links = Links.new(attrs.fetch("_links"))
  end
end
```

#### Loaders
A loader is an object that loads up resources for me, going through any of the hypermedia bits that might be necessary. For example I might have a loader that gets a list of orders.

```ruby
class OrdersLoader < Struct.new(:user)
  def load
    client.basic_auth(user.email, user.password)

    response = client.get("/")
    root = Root.load(response.body)

    response = client.get(root.links.fetch("orders"))
    Orders.load(response.body)
  end
end
```

In the end I was pretty happy with how this structure ended up and was able to fairly quickly come up with an app that used it.
