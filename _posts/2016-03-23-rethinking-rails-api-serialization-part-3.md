---
layout: post
categories:
- ruby
- rails
- api
- serialization
title: Rethinking Rails API Serialization - Part 3
---

Part 3 will wrap up this series and show off a full example of how [Nagare][nagare] can be used.

Name fun fact: Nagare (流れ) is Japanese for a stream or flow.

## Railtie

The railtie included with Nagare changes the default `_render_with_renderer_json` method on controllers, similar to [ActiveModel::Serializers][ams]. You can see a full version of it [on github][nagarerailtie].

## A Full Example

### Controllers

We start by defining a context that lets the context have a current user. Anything else a serializer might want can be added here.

```ruby
class ApplicationController < ActionController::Base
  private

  def nagare_context
    @nagare_context ||= Nagare::Context.new({
      current_user: current_user,
    })
  end
end
```

A regular controller looks like this:

```ruby
class AdminOrdersController < ApplicationController
  def index
    render({
      json: orders,
      serializers: { collection: AdminOrdersSerializer, item: AdminOrderSerializer },
      context: {
        href: orders_url,
      },
    })
  end

  def show
    render({
      json: order,
      serializers: { item: AdminOrderSerializer },
      context: {
        href: order_url(order),
      },
    })
  end
end
```

The collection key is required if you have a collection. I found this to be a good pattern when working on [Artifact][artifact].

### Serializers

The serializers are pretty simple. The serializers can customize which attributes will be output. In Artifact I extend these with a custom DSL for [Collection+JSON][cjson] to include links, queries, and templates.

```ruby
class AdminOrdersSerializer < Nagare::Collection
  # This is the key that will contain all of the serialized items.
  key "orders"

  # You can also have extra attributes on the collection
  attributes :count

  def count
    collection.count
  end
end
```

Serializers have several ways of obtaining attributes. It can be a method on the serializer itself, the object, or from the context.

```ruby
class AdminOrderSerializer < Nagare::Item
  # email is a method off of the order
  # item_count is a method we define
  # href comes from the passed in context
  attributes :email, :item_count, :href

  def item_count
    object.items.count
  end
end
```

`attributes` defines a method per attribute that will try the object or context for the attribute value. Otherwise you can simply define your own method to use.

## Wrapping Up

This brings the Rethinking Rails API Serialization series to a close. I hope you found something useful. Please read over the source for Nagare. I welcome issues or pull requests.

### Rethinking Rails API Serializations Series

- [Part 1][part1]
- [Part 2][part2]
- [Part 3][part3]

[nagare]: https://github.com/oestrich/nagare
[ams]: https://github.com/rails-api/active_model_serializers
[nagarerailtie]: https://github.com/oestrich/nagare/blob/a8d149cd87367a65e08f59cefa57318e6485895a/lib/nagare/railtie.rb
[artifact]: https://www.discoverartifacts.com/
[part1]: {% post_url 2016-03-21-rethinking-rails-api-serialization-part-1 %}
[part2]: {% post_url 2016-03-22-rethinking-rails-api-serialization-part-2 %}
[part3]: {% post_url 2016-03-23-rethinking-rails-api-serialization-part-3 %}
