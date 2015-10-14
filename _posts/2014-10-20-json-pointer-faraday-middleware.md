---
layout: post
categories:
- ruby
- faraday
- json
title: JSON Pointer Faraday Middleware
---

At [Ruby DCamp][rubydcamp] I spent some time writing a faraday middleware that  automatically dereferences JSON pointers. It was a fun exercise in reading an [RFC spec][rfc] and writing something that implements it. I wanted to do this after some cool demonstrations I saw at RESTFest this year. One was [hyper+json][hyperjson] which uses JSON Pointers.

Below is what I came up with. It is also hosted on  [github][faraday_json_pointers]. The github project also contains specs which better highlight what is happening.

This code will take the following JSON:

```json
{
  "name": "/first_name",
  "first_name": "Eric",
}
```

And convert it into:

```json
{
  "name": "Eric",
  "first_name": "Eric",
}
```

It even allows deep linking of attributes.


##### json_faraday_middleware.rb

```ruby
require 'faraday'
require 'json'

class JsonPointerMiddleware < Faraday::Middleware
  JSON_MIME_TYPE = "application/json"

  def call(env)
    @app.call(env).on_complete do |env|
      unless env.response_headers["Content-Type"] == JSON_MIME_TYPE
        return
      end

      @body = JSON.parse(env.body)
      pointerize(@body)
      begin
        env.body = @body.to_json
      rescue JSON::NestingError
        # there is a circular nest, skip dereferencing
      end
    end
  end

  def pointerize(body)
    body.each do |key, value|
      if value.is_a?(Hash)
        pointerize(value)
      end
      next unless value =~ /\//

      pointer_keys = value.split("/")[1..-1]
      pointer_keys = escape_slash(pointer_keys)
      pointer_keys = escape_tilde(pointer_keys)
      pointer_keys = convert_indices(pointer_keys)
      new_value = pointer_keys.inject(@body) do |body, pointer_key|
        next if body.nil?
        body[pointer_key]
      end
      body[key] = new_value if new_value
    end
  end

  def escape_slash(keys)
    keys.map do |key|
      key.gsub("~1", "/")
    end
  end

  def escape_tilde(keys)
    keys.map do |key|
      key.gsub("~0", "~")
    end
  end

  # Convert array indices to Integers
  def convert_indices(keys)
    keys.map do |key|
      Integer(key) rescue key
    end
  end
end
```

[rubydcamp]: http://rubydcamp.org/
[hyperjson]: https://github.com/hypergroup/hyper-json
[rfc]: http://tools.ietf.org/html/rfc6901
[faraday_json_pointers]: https://github.com/oestrich/faraday_json_pointers
