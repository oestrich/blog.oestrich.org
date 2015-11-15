---
layout: post
categories:
- elasticsearch
- rails
title: Elasticsearch Proxy Rack App
description: A basic proxy app for Elasticsearch
date: 2015-11-15 1:30 pm
---

For a project at work we use Elasticsearch. It was hosted inside of Kubernetes so we didn't have a nice way to access the production instance from our browsers. I came up with this small proxy rack app that fowards the request through the Rails app. I have it authenticated via devise so only admins can view the actual instance.

## The Rack App

```ruby
class ElasticsearchForwarder
  def self.call(env)
    new.call(env)
  end

  def call(env)
    request = Rack::Request.new(env)

    case request.request_method
    when "GET", "POST", "OPTIONS", "HEAD", "PUT", "DELETE"
      response = client.send(request.request_method.downcase, request.path_info) do |f|
        f.params = request.params
        f.body = request.body.read
      end

      [response.status, headers(response.headers), [response.body]]
    else
      [405, {}, []]
    end
  end

  private

  def headers(headers)
    cors_headers.merge(headers)
  end

  def cors_headers
    {
      "Access-Control-Allow-Origin" => "*",
      "Access-Control-Allow-Methods" => "OPTIONS, HEAD, GET, POST, PUT, DELETE",
      "Access-Control-Allow-Headers" => "X-Requested-With, Content-Type, Content-Length",
      "Access-Control-Max-Age" => "1728000",
    }
  end

  def client
    @client ||= Faraday.new(AppContainer.elasticsearch_url) do |conn|
      conn.adapter :net_http
    end
  end
end
```

Make sure to replace `AppContainer.elasticsearch_url` with your own elasticsearch instance information. Otherwise it is a pretty simple rack app that just uses Faraday to forward your request on to elasticsearch. I set up CORS headers to allow anything, but I found with using the devise authentication it required that you were on the same domain anyways.

## Mounting the App

```ruby
authenticate :user, lambda { |user| user.admin? } do
	mount ElasticsearchForwarder => "/elasticsearch"
end
```

Here we mount the app inside of a devise `authenticate` block, making sure the user is an admin and logged in.

## Future Improvements

I would like to make this less reliant on the devise cookie. We had to host our own ElasticHQ in order for devise to let requests through. It's not that bad, but would have been cleaner to not require hosting our own.
