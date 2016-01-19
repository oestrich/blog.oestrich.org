---
layout: post
categories:
- ruby
- redis
- google cloud storage
title: Redis Cache for Google Cloud Storage Access
description: Add a simple redis read cache to speed up Google Cloud Storage access
date: 2016-01-19 10:30 AM
---

In my side project, [worfcam][worfcam], I host photos on [Google Cloud Storage][gcs]. When viewing pages that had multiple thumbnails on them I noticed loading photos took a while. Each photo took 400-500ms which isn't bad but comes with a noticeable flicker of them loading. My application serves photos out, users do not go directly to GCS to load them. Since users are coming to my application, I was able to add a simple caching layer in front of the ruby GCS service.

This is a cache layer above the [google-ruby-api-client][grac] gem.

##### gcs_service_cacher.rb

```ruby
class GcsServiceCacher
  SIXTY_MINUTES = 60 * 60

  def initialize(service, redis_pool)
    @service = service
    @redis_pool = redis_pool # This is a ConnectionPool
  end

  def method_missing(method, *args)
    if @service.respond_to?(method)
      @service.send(method, *args)
    else
      super
    end
  end

  def insert_object(bucket, name:, upload_source:, cache: false)
    if cache
      @redis_pool.with do |redis|
        redis.setex("photo-cache:#{name}:binary", SIXTY_MINUTES, upload_source.read)
      end

      upload_source.rewind
    end

    @service.insert_object(bucket, name: name, upload_source: upload_source)
  end

  def get_object(bucket, path, download_dest:, cache: false)
    if cache
      data = @redis_pool.with do |redis|
        redis.get("photo-cache:#{path}:binary")
      end

      return StringIO.new(data) unless data.nil?
    end

    body = @service.get_object(bucket, path, download_dest: download_dest)
    body.rewind

    if cache
      @redis_pool.with do |redis|
        redis.setex("photo-cache:#{path}:binary", SIXTY_MINUTES, body.read)
      end
      body.rewind
    end

    body
  end
end
```

This is a pretty simple class. The two important methods are `#insert_object` and `#get_obejct`, everything else is delegated down to the actual service class via `#method_missing`. The `redis_pool` object is a [ConnectionPool][connectionpool] instance loaded with Redis.

The `#insert_method` class just sets the `upload_source` binary data to a redis key and delegates down. I have photos expire after 60 minutes to not clog up redis with old photos.

It should be noted that these two methods keep the same method signature as the service class itself, except for the `cache` keyword. I needed a way to let objects above this one hint that a file should not be cached. I only wanted to cache small thumbnails in redis to not completely blow away the memory. Thumbnails are the most accessed file as well.

The `#get_object` class is a little more complicated. If cacheing is enabled, it will check redis to see if the key exists and return that if it does. Otherwise it will pull it down from the service and cache it for quicker access next read.

## Final results

With the caching layer in place, I have noticed extremely quick reads compared to loading from GCS each page load. Photo loading dropped to about 100-200ms total. Adding in correct caching headers completely reduces load time as well.

A nice side benefit to this is potentially reducing GCS bandwidth. Worfcam is completely hosted inside of the Google Cloud so I don't have bandwidth fees between the Compute Engine and Cloud Storage, but if you weren't hosted inside of Compute Engine then you could read from your caching layer to prevent huge bandwidth bills. Worfcam pulls down each file at least once and rolls every hour into a gif so this would help me out if I ever decide to host elsewhere.

[worfcam]: https://worfcam.com/
[gcs]: https://cloud.google.com/storage/
[grac]: https://github.com/google/google-api-ruby-client
[connectionpool]: https://github.com/mperham/connection_pool
