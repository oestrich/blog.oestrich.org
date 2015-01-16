---
layout: post
categories:
- ruby
- faraday
- web
title: Simple Ruby Faraday Pager Class
---

This is a fairly simple pager I wrote for a project recently. It's not the exact one as it needed more features that a generic pager doesn't. This should cover most cases pretty nicely.

One good feature about this is that it yields items as it goes instead of collecting them up for the end. This lets you do work in-between requests hopefully spacing them out a bit, and not completely slamming the service you're paging from.

I also like that you can completely ignore the HTTP interactions and just deal with regular ruby `Enumerable` methods.

{% highlight ruby %}
class Pager < Struct.new(:client, :path, :json_key)
  include Enumerable

  def each
    fetch(1) do |item|
      yield item
    end
  end

  private

  def fetch(page_number, &block)
    response = client.get(path) do |req|
      req.params["page"] = page_number
    end
    items = JSON.parse(response.body).fetch(json_key, [])
    items.each do |item|
      yield item
    end
    if items.count > 0
      fetch(page_number + 1, &block)
    end
  end
end
{% endhighlight %}
