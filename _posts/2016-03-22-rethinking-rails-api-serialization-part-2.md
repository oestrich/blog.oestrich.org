---
layout: post
categories:
- ruby
- rails
- api
- serialization
title: Rethinking Rails API Serialization - Part 2
description: Common problems I had with Rails API serialization - Part 2 How to deal with them
date: 2016-03-22 10:00 AM
---

Part 2 of this series will show off snippets of a new gem I created, [Nagare][nagare]. The gem is pretty small so give it a read through. Part 3 will show off more of the gem, this post will concentrate on the specific issues I had in the first post.

## General Context

The first thing I wanted to tackle is injecting context to the serializer. I do this by making you specify everything you want the serializer to know about. The default context has nothing.

```ruby
class ApplicationController < ActionController::Base
  def nagare_context
    @nagare_context ||= Nagare::Context.new({
      current_user: current_user,
    })
  end
end
```

It's simple to create a new context and you can even override it in specific controllers because it's just a method. I tend to use my [letter][letter] gem here to handle memoization for me.

## Resource Context

Next up is context on the resource level. This is similar to how I injected context by [ActiveModel::Serializers][ams], the biggest difference is the key change.

```ruby
def index
  render({
    context: {
      href: books_url,
    },
  })
end
```

This hash [extends the general context][extendcontext] so you can easily override specific keys if necessary. 

## Collection Serializers

Collection serializers are a subclass of an item serializer in Nagare. This gives them the full capability of `attribute` and the context.

```ruby
class BooksSerializer < Nagare::Collection
  attribute :count, :href

  def count
    collection.count
  end
end
```

`href` is coming from the context, if we think of this rendering in the `index` method shown above.

## Explicit Resources

Nagare requires you to explicitly set the serializer for it to do anything. This is how I prefer it because I almost never tended to use the automatic serializer choice.

```ruby
def index
  render({
    serializers: { collection: BooksSerializer, item: BookSerializer },
  })
end
```

## Adapters

This was one thing I really liked from ActiveModel::Serializers and copied for Nagare. ActiveModel::Serializers lets you define an adapter that can completely reshape the JSON before it leaves your app. I used this for [Collection+JSON with ActiveModel::Serializers][cjsonams].

Nagare's version of adapters is pretty simple to start. The only interface is `#as_json`. Here is the full default adapter:

```ruby
class Adapter
  def initialize(serializer, collection: false)
    @serializer = serializer
    @collection = collection
  end

  def as_json(options = nil)
    serializer.as_json(options)
  end

  private

  attr_reader :serializer, :collection
end
```

This is pretty simple, but you have a hook into doing more complicated things like Collection+JSON or JSON API.

## Part 3

Next post I'll show off how to use Nagare further.

### Rethinking Rails API Serializations Series

- [Part 1][part1]
- [Part 2][part2]
- [Part 3][part3]

[nagare]: https://github.com/oestrich/nagare
[letter]: https://github.com/oestrich/letter
[ams]: https://github.com/rails-api/active_model_serializers
[extendcontext]: https://github.com/oestrich/nagare/blob/b693e61bc9f4662f51b82c8149f9da4c4a802a73/lib/nagare.rb#L114
[cjsonams]: {% post_url 2015-09-29-active-model-serializers-collection-json %}
[part1]: {% post_url 2016-03-21-rethinking-rails-api-serialization-part-1 %}
[part2]: {% post_url 2016-03-22-rethinking-rails-api-serialization-part-2 %}
[part3]: {% post_url 2016-03-23-rethinking-rails-api-serialization-part-3 %}
