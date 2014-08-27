---
layout: post
categories:
- rails
- ruby
title: cURLin' for Docs
---

_This post was originally published on the_
[SmartLogic Blog](http://blog.smartlogicsolutions.com/2012/07/12/curlin-for-docs/).

You may have docs for your API, but do you have an API for your docs? With RspecApiDocumentation and Raddocs, you can cURL for your documentation. Try out the following cURL command to see for yourself:

{% highlight bash %}
$ curl -H "Accept: text/docs+plain" http://rad-example.herokuapp.com/orders
{% endhighlight %}

![This cURL trick might save you so much time you can go curling! (Sorry.)](/images/curling.jpeg)

1.  Install the gem
1.  Configure
1.  Write tests
1.  Host via public or Raddocs

Also available as a [presentation](http://oestri.ch/presentations/cURLin_for_docs.pdf).

## What is the rspec_api_documentation gem?

The rspec_api_documentation gem (RAD for short) is a gem that lets you easily create documentation for your API based on tests for the API. You write acceptance tests and it will output docs.

## Assumptions

*  rspec is installed and setup

## Installing the gem

Installing RAD is as simple as adding it to your Gemfile. You might want to add it to a test and development group because it has a handy rake task for generating docs.

##### Gemfile
{% highlight ruby %}
gem "rspec_api_documentation"
{% endhighlight %}

##### Install gem
{% highlight bash %}
$ bundle
{% endhighlight %}

## Configure

Once RAD is installed there are several options you probably want to configure. Listed below are all of the default options that RAD ships with. The most common ones you will want to change are "api_name", "docs_dir", "format", and "url_prefix".

##### spec/spec_helper.rb
{% highlight ruby %}
RspecApiDocumentation.configure do |config|
  # All settings shown are the default values
  config.app = Rails.application # if it’s a rails app
  config.api_name = “API Documentation”
  config.docs_dir = Rails.root.join(“docs”)
  config.format = :html # also: :json, :wurl, :combined_text, :combined_json
  config.url_prefix = “”

  # If you want cURL commands to be included in your docs,
  # set to not nil
  config.curl_host = nil # “http://myapp.example.com”

  # Filtering
  # If you set the :document key on an example to a particular group
  # you can only output those examples
  config.filter = :all

  # You can also exclude by keys
  config.exclusion_filter = nil

  # To use your own templates
  config.template_path = “” # The default template path is inside of RAD

  # Instead of sorting alphabetically, keep the order in the spec file
  config.keep_source_order = false

  config.define_group :public do |config|
    # When you define a sub group these defaults are set
    # Along with all of the parents settings
    config.docs_dir = Rails.root.join(“docs”, “public”)
    config.filter = :public
    config.url_prefix = “/public”
  end
end
{% endhighlight %}

## Write tests

Tests for RAD are written in a DSL which helps assist in getting the metadata correct for properly formatting the outputted docs. Tests go in `spec/acceptance`.

##### spec/acceptance/orders_spec.rb
{% highlight ruby %}
require 'acceptance_helper'

resource "Orders" do
  header "Accept", "application/json"
  header "Content-Type", "application/json"

  let(:order) { Order.create(:name => "Old Name", :paid => true, :email => "email@example.com") }

  get "/orders" do
    parameter :page, "Current page of orders"

    let(:page) { 1 }

    before do
      2.times do |i|
        Order.create(:name => "Order #{i}", :email => "email#{i}@example.com", :paid => true)
      end
    end

    example_request "Getting a list of orders" do
      expect(response_body).to eq(Order.all.to_json)
      expect(status).to eq(200)
    end
  end

  head "/orders" do
    example_request "Getting the headers" do
      expect(response_headers["Cache-Control"]).to eq("no-cache")
    end
  end

  post "/orders" do
    parameter :name, "Name of order", :required => true, :scope => :order
    parameter :paid, "If the order has been paid for", :required => true, :scope => :order
    parameter :email, "Email of user that placed the order", :scope => :order

    response_field :name, "Name of order", :scope => :order, "Type" => "String"
    response_field :paid, "If the order has been paid for", :scope => :order, "Type" => "Boolean"
    response_field :email, "Email of user that placed the order", :scope => :order, "Type" => "String"

    let(:name) { "Order 1" }
    let(:paid) { true }
    let(:email) { "email@example.com" }

    let(:raw_post) { params.to_json }

    example_request "Creating an order" do
      explanation "First, create an order, then make a later request to get it back"

      order = JSON.parse(response_body)
      expect(order.except("id", "created_at", "updated_at")).to eq({
        "name" => name,
        "paid" => paid,
        "email" => email,
      })
      expect(status).to eq(201)

      client.get(URI.parse(response_headers["location"]).path, {}, headers)
      expect(status).to eq(200)
    end
  end

  get "/orders/:id" do
    let(:id) { order.id }

    example_request "Getting a specific order" do
      expect(response_body).to eq(order.to_json)
      expect(status).to eq(200)
    end
  end

  put "/orders/:id" do
    parameter :name, "Name of order", :scope => :order
    parameter :paid, "If the order has been paid for", :scope => :order
    parameter :email, "Email of user that placed the order", :scope => :order

    let(:id) { order.id }
    let(:name) { "Updated Name" }

    let(:raw_post) { params.to_json }

    example_request "Updating an order" do
      expect(status).to eq(204)
    end
  end

  delete "/orders/:id" do
    let(:id) { order.id }

    example_request "Deleting an order" do
      expect(status).to eq(204)
    end
  end
end
{% endhighlight %}

### DSL Methods of Interest

See [https://github.com/zipmark/rspec_api_documentation/wiki/DSL](https://github.com/zipmark/rspec_api_documentation/wiki/DSL)

## Host via public or Raddocs
### Public

This is the easiest method. If you generate HTML or wURL HTML output then you can simply place the generated docs inside of the publicly accessible folder in your application when you deploy.

### Raddocs

Raddocs is a simple Sinatra app that will take the JSON output from RAD and serve it up as HTML pages. The output is very similar to the HTML generated pages, but Raddocs allows us to have better asset handling than straight HTML.

1.  Generate :json and :combined_text output from RAD
1.  Configure Raddocs
1.  Mount Raddocs

##### spec/spec_helper.rb
{% highlight ruby %}
RspecApiDocumentation.configure do |config|
  config.formats = [:json, :combined_text]
end
{% endhighlight %}

##### config/initializers/raddocs.rb
{% highlight ruby %}
Raddocs.configure do |config|
  # output dir from RAD
  config.docs_dir = "docs"

  # Should be in the form of text/vnd.com.example.docs+plain
  config.docs_mime_type = /text\/docs\+plain/
end
{% endhighlight %}

##### config/routes.rb
{% highlight ruby %}
match "/docs" => Raddocs::App, :anchor => false
{% endhighlight %}

For middleware

##### config/application.rb
{% highlight ruby %}
config.middleware.use "Raddocs::Middleware"
{% endhighlight %}

## Conclusion

You now have docs that won't generate if your tests fail, making sure that they are correct. And you can view them in a console as well as the browser.

[Image Source](http://www.flickr.com/photos/rtclauss/7200798740/)
