---
layout: post
categories:
- ruby
- rails
- api
- serialization
title: Rethinking Rails API Serialization - Part 1
description: Common problems I had with Rails API serialization
date: 2016-03-21 10:00 AM
---

For the last few months I've been working on the API for [Artifact][artifact]. I started out the project using [ActiveModel::Serializers][ams] and chose [collection+JSON][cjson] as the media type. I wanted to give collection+json a try after using it at [REST Fest 2015][restfest]. This series of posts will show not why ActiveModel::Serializers is bad, but bad for what I tried using it for.

## AMS + JSON API

ActiveModel::Serializer is heavily geared towards using JSON API. Because of this it feels like I've had to bend backwards to get it to do collection+json. You can see what I did in my [Collection+JSON with ActiveModel::Serializers][cjsonams] post.

I use and abuse the `meta` key in that post. Based on what I saw in the source code for ActiveModel::Serializer I don't think I used the key as intended. While I misuse it, it was the only way I could find to get variables into the serializer from the `render` method.

## Serializer Context

Another issue I ran across was trying to get general context inside of a serializer. ActiveModel::Serializer does `current_user` for you, extending this to include extra methods resulted in a fairly hacky monkey patch.

```ruby
class ApplicationController < ActionController::Base
  # Extend what the Serializer can see by injecting the
  # current application.
  def _render_with_renderer_json(resource, options)
    options[:current_application] = current_application
    super(resource, options)
  end
end
```

This is what I added in order to push the current OAuth application inside of the serializer. I found this by browsing the source of ActiveModel::Serializer enough to finally see them do it for `current_user`. Inside of the serializer this is required:

```ruby
def current_application
  @instance_options[:current_application]
end
```

It took a good deal of poking around in order to figure out how to inject this extra context into the serializer.

## Collection Serializers

Collection serializers are second citizens in ActiveModel::Serializers. With collection+json, everything is a collection. Every resource I have in Artifact has an item and a collection serializer. While writing the collection serializers you start to notice things that are missing in the `ActiveModel::Serializer::ArraySerializer`. A lot of the context isn't present and you see missing method errors more than you would expect.

## Implicit Resources

One thing that seems like a nice feature of ActiveModel::Serializers is automatically detecting the serializer class. In practice I rarely use this feature. I have started treating API endpoints as resources and typically create a special serializer for the one endpoint. I found that I don't create just a `BookSerializer` and use it everywhere.

For example, in Artifact you can create a collection of books. For this I would create a `CollectionBookSerializer` because the URLs contained within will be different than just a `BookSerializer`. I would also need the associated collection serializers.

Given this here is how a typical controller method renders:

```ruby
def index
  render({
    :json => books,
    :each_serializer => CollectionBookSerializer,
    :serializer => CollectionBooksSerializer,
    :meta => { }, # "..."
  })
end
```

I personally now prefer explicitly referring to a serializer I want to use on an endpoint.

## Next Time

These are all fairly minor pain points, but it has recently gotten me to thinking how I can make it better. In the next post I'll go over what I came up with.

### Rethinking Rails API Serializations Series

- [Part 1][part1]
- [Part 2][part2]
- [Part 3][part3]

[artifact]: https://www.discoverartifacts.com/
[ams]: https://github.com/rails-api/active_model_serializers
[cjson]: http://amundsen.com/media-types/collection/
[restfest]: http://restfest.org/
[cjsonams]: {% post_url 2015-09-29-active-model-serializers-collection-json %}
[part1]: {% post_url 2016-03-21-rethinking-rails-api-serialization-part-1 %}
[part2]: {% post_url 2016-03-22-rethinking-rails-api-serialization-part-2 %}
[part3]: {% post_url 2016-03-23-rethinking-rails-api-serialization-part-3 %}
