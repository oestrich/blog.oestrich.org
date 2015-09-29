---
layout: post
categories:
- rails
- collection+json
- api
- activemodel serializers
title: Collection+JSON with ActiveModel::Serializers
description: How to use Collection+JSON and ActiveModel::Serializers together
date: 2015-09-29 12:00 pm
---

For work I'm starting on a new API and I decided to use [Collection+JSON][collectionjson] after using it at [REST Fest][restfest]. I enjoyed making a ruby client for it, so I figured I would see how it works further than a weekend.
 
I like to use [ActiveModel::Serializers][activemodelserializers] and this DSL has made generating Collection+JSON extremely simple. We are using ActiveModel::Serializers master branch on github. Not the gem currently on Rubygems so you may need to tweak some things to get it working on the older version.

## Using The DSL

##### app/controllers/orders_controller.rb

``` ruby
class OrdersController < ApplicationController
  def index
    render :json => orders, :serializer => OrdersSerializer, :meta => {
      :href => orders_url(params.slice(:search, :page, :per)),
      :up => root_url,
      :search => params[:search] || "",
    }
  end

  def show
    render :json => [order], :serializer => OrdersSerializer, :meta => {
      :href => order_url(order.first),
      :up => orders_url,
      :search => "",
    }
  end
end
```

Here is the controller. Of note is `show` displaying an array of orders, since there is no single element in Collection+JSON. This was a lot nicer than I expected, it flowed very well.

##### app/serializers/order_serializer.rb

``` ruby
class OrderSerializer < ActiveModel::Serialier
  include CollectionJson

  attributes :name, :ordered_at

  href do
    meta(:href)
  end
end
```

Here is the `OrderSerializer`. It includes `CollectionJson` which adds the DSL. The important feature is `href`. I found that passing in the `href` from the controller is best. I pull the `href` out from the `meta` hash. I'm not super sure this is the right way to go, but it's worked well so far.

##### app/serializers/orders_serializer.rb

``` ruby
class OrdersSerializer < ActiveModel::Serializer
  include CollectionJson

  link "up" do
    meta(:up)
  end

  query "http://example.com/rels/order-search" do
    href orders_url
    data :search, :value => meta(:search), :prompt => "Order Name"
  end

  template do
    data "name", :value => "", :prompt => "Order Name"
  end

  href do
    meta(:href)
  end

  private

  def include_template?
    true
  end
end
```

Here we see some more of the DSL. Generating links, queries, and the template. You can see more about these in the code below.

##### config/initializers/serializers.rb

``` ruby
require 'collection_json'
ActiveModel::Serializer.config.adapter = :collection_json

class ActiveModel::Serializer::ArraySerializer
  include Rails.application.routes.url_helpers
  delegate :default_url_options, :to => "ActionController::Base"
end

class ActiveModel::Serializer
  include Rails.application.routes.url_helpers
  delegate :default_url_options, :to => "ActionController::Base"
end
```

Here we set the adapter to `collection_json` and include routes so the serializers can be hypermedia driven. I'm not super stoked about how many times we need to include routes, but I couldn't figure out a way around it.

## The Details

Here are the two files that make the above happen:

##### app/serializers/collection_json.rb

``` ruby
module CollectionJson
  extend ActiveSupport::Concern

  module RoutesHelpers
    include Rails.application.routes.url_helpers
    delegate :default_url_options, :to => "ActionController::Base"
  end

  module CollectionObject
    def encoding(encoding)
      @encoding = encoding
    end

    def data(name, options = {})
      @data << options.merge({
        :name => name,
      })
    end

    def compile(metadata)
      reset_query
      @meta = metadata
      instance_eval(&@block)
      generate_json
    end

    private

    def meta(key)
      @meta && @meta[key]
    end
  end

  class Query
    include CollectionObject
    include RoutesHelpers

    def initialize(rel, &block)
      @rel = rel
      @block = block
    end

    def href(href)
      @href = href
    end

    private

    def generate_json
      json = {
        :rel => @rel,
        :href => CGI.unescape(@href),
        :data => @data,
      }
      json[:encoding] = @encoding if @encoding
      json
    end

    def reset_query
      @data = []
      @encoding = nil
      @href = nil
    end
  end

  class Template
    include CollectionObject
    include RoutesHelpers

    def initialize(&block)
      @block = block
    end

    private

    def generate_json
      json = { :data => @data }
      json[:encoding] = @encoding if @encoding
      json
    end

    def reset_query
      @data = []
      @encoding = nil
    end
  end

  class_methods do
    include RoutesHelpers

    def link(rel, &block)
      @links ||= []
      @links << {
        :rel => rel,
        :href => block,
      }
    end

    def links
      @links
    end

    def query(rel, &block)
      @queries ||= []
      @queries << Query.new(rel, &block)
    end

    def queries
      @queries
    end

    def template(&block)
      return @template unless block_given?
      @template = Template.new(&block)
    end

    def href(&block)
      define_method(:href) do
        instance_eval(&block)
      end
    end
  end

  def links
    return unless self.class.links.present?
    self.class.links.map do |link|
      link.merge({
        :href => instance_eval(&link[:href]),
      })
    end.select do |link|
      link[:href].present?
    end
  end

  def queries
    return unless self.class.queries.present?
    self.class.queries.map do |query|
      query.compile(@meta)
    end
  end

  def template
    if respond_to?(:include_template?, true) && include_template?
      self.class.template.compile(@meta) if self.class.template.present?
    end
  end

  def meta(key)
    @meta && @meta[key]
  end
end
```

##### lib/collection_json.rb

``` ruby
module ActiveModel
  class Serializer
    module Adapter
      class CollectionJson < Base
        def initialize(serializer, options = {})
          super
          @hash = {}
        end

        def serializable_hash(options = {})
          @hash[:version] = "1.0"
          @hash[:href] = serializer.href
          @hash[:links] = serializer.links if serializer.links.present?
          @hash[:queries] = serializer.queries if serializer.queries.present?
          @hash[:template] = serializer.template if serializer.template.present?

          add_items

          {
            collection: @hash,
          }
        end

        def as_json(options = nil)
          serializable_hash(options)
        end

        private

        def add_items
          if serializer.respond_to?(:each)
            serializer.map do |serializer|
              add_item(serializer)
            end
          else
            add_item(serializer) if serializer.object
          end
        end

        def add_item(serializer)
          @hash[:items] ||= []

          item = {}
          item[:href] = serializer.href
          item[:links] = serializer.links if serializer.links.present?

          serializer.attributes.each do |key, value|
            next unless value.present?
            item[:data] ||= []

            data = {
              name: key.to_s,
              value: value,
            }

            item[:data] << data
          end if serializer.attributes.present?

          @hash[:items] << item
        end
      end
    end
  end
end
```

[collectionjson]: http://amundsen.com/media-types/collection/
[activemodelserializers]: https://github.com/rails-api/active_model_serializers
[restfest]: http://www.restfest.org/
