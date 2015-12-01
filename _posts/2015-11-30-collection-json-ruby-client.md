---
layout: post
categories:
- ruby
- hypermedia
- colection+json
title: Collection+JSON Ruby Client
description: Building a Collection+JSON client in ruby
date: 2015-11-30 10:00 PM
---

At [REST Fest][restfest] this year I did a small [Collection+JSON][collectionjson] ruby client to work against a hypermedia server. The full source code is [hosted on github][cj-source].

## Client classes

```ruby
class CollectionJSONMiddleware < Faraday::Middleware
  def initialize(app)
    @app = app
  end

  def call(env)
    env[:request_headers]["Accept"] = "application/vnd.collection+json"
    @app.call(env)
  end
end
```

A small middleware class to shove the correct accept header in.

```ruby
class Client
  def initialize(url)
    @url = url
  end

  def get(url)
    response = connection.get(url)
    json = JSON.parse(response.body)
    Collection.new(json)
  end

  def delete(item)
    response = connection.delete(item.href)
    json = JSON.parse(response.body)
    Collection.new(json)
  end

  def update(item, template)
    response = connection.put(item.href) do |req|
      req.headers["Content-Type"] = "application/vnd.collection+json"
      req.body = template.to_json
    end
    json = JSON.parse(response.body)
    Collection.new(json)
  end

  def add_template(collection, template)
    response = connection.post(collection.href) do |req|
      req.headers["Content-Type"] = "application/vnd.collection+json"
      req.body = template.to_json
    end
    json = JSON.parse(response.body)
    Collection.new(json)
  end

  private

  def connection
    @connection ||= Faraday.new(@url) do |conn|
      conn.use CollectionJSONMiddleware
      conn.adapter Faraday.default_adapter
    end
  end
end
```

## Model Classes

These are Collection+JSON classes that are not specific to the underlying data at all.

```ruby
class Collection
  attr_reader :version, :href, :links, :items, :queries, :template

  def initialize(json)
    collection = json.fetch("collection")

    @version = collection.fetch("version", nil)
    @href = collection.fetch("href", nil)
    @links = collection.fetch("links", []).map { |link| Link.new(link) }
    @items = collection.fetch("items", []).map { |item| Item.new(item) }
    @queries = collection.fetch("queries", []).map do |query|
      Query.new(query)
    end
    template = collection.fetch("template", nil)
    @template = Template.new(template) if template
  end

  def query(rel)
    queries.detect do |query|
      query.rel == rel
    end
  end
end

class Query
  attr_reader :rel, :href, :prompt, :data

  def initialize(attributes)
    @rel = attributes.fetch("rel")
    @href = attributes.fetch("href")
    @prompt = attributes.fetch("prompt")
    @data = attributes.fetch("data", []).map do |data|
      DataItem.new(data)
    end
  end

  def set(data, value)
    @values ||= Faraday::Utils::ParamsHash.new
    @values[data.name] = value
  end

  def to_url
    uri = URI.parse(href)
    uri.query = @values.to_query
    uri.to_s
  end
end

class Template
  attr_reader :prompt, :rel, :data

  def initialize(attributes)
    @prompt = attributes.fetch("prompt", nil)
    @data = attributes.fetch("data", []).map do |data|
      DataItem.new(data)
    end
  end

  def set(data, value)
    @values ||= {}
    @values[data.name] = value
  end

  def to_json
    {
      :template => {
        :data => data.map do |data|
          {
            :name => data.name,
            :value => @values[data.name],
          }
        end
      },
    }.to_json
  end
end

class Link
  def initialize(attributes)
    @attributes = attributes
  end

  def to_s
    "#{rel}: #{href}"
  end

  def [](key)
    @attributes[key]
  end

  def rel
    @attributes.fetch("rel")
  end

  def href
    @attributes.fetch("href")
  end
end

class Item
  attr_reader :href, :data, :links

  def initialize(attributes)
    @href = attributes.fetch("href")
    @data = attributes.fetch("data", []).map do |data|
      DataItem.new(data)
    end
    @links = attributes.fetch("links", []).map { |link| Link.new(link) }
  end

  def attribute(name)
    data.detect { |attribute| attribute.name == name }
  end
end

class DataItem
  def initialize(attributes)
    @attributes = attributes
  end

  def to_s
    "#{name}: #{value}"
  end

  def [](key)
    @attributes[key]
  end

  def name
    @attributes.fetch("name")
  end

  def value
    @attributes.fetch("value", nil)
  end

  def prompt
    @attributes.fetch("prompt", nil)
  end
end
```

## Putting it together

Here is a class that lets you perform general collection+json actions on the command line. It is slightly specific for the app at REST Fest, but there isn't that much specific.

```ruby
class CommandLine
  attr_reader :collection

  def run
    @collection ||= client.get("/")

    puts "What do you want to do?"
    puts "  - Refresh Items (refresh)"
    puts "  - Items (items)"
    puts "  - Queries (queries)"
    puts "  - Template '#{collection.template.prompt}' (template)" if collection.template
    puts "  - Delete (delete)"
    puts "  - Edit (edit)"
    puts "  - Exit (exit)"

    choice = gets.chomp
    puts

    case choice
    when "refresh"
      refresh
    when "items"
      print_items(collection.items)
    when "queries"
      queries
    when "template"
      template
    when "delete"
      delete
    when "edit"
      edit
    when "exit"
      return
    end

    run
  end

  def refresh
    @collection = client.get(collection.href)
  end

  def queries
    queries = collection.queries.map do |query|
      "#{query.prompt} (#{query.rel})"
    end
    puts queries

    puts "Choose a query"
    query = gets.chomp

    search_query = collection.query(query)

    unless search_query
      puts "Query not found"
      return
    end

    puts

    if search_query.data.all? { |data| data.value.empty? }
      edit = "yes"
    else
      search_query.data.each do |data|
        puts "#{data.prompt}: #{data.value}"
      end

      puts "Edit? (yes/no)"
      edit = gets.chomp
      puts
    end

    case edit
    when "yes"
      search_query.data.each do |data|
        if data.value.present?
          puts "#{data.prompt} (#{data.value}):"
        else
          puts "#{data.prompt}:"
        end
        search_query.set(data, gets.chomp)
      end
      puts
    when "no"
      search_query.data.each do |data|
        search_query.set(data, data.value)
      end
    end

    collection = client.get(search_query.to_url)

    print_items(collection.items)
  end

  def template
    template = collection.template
    puts template.prompt
    template.data.each do |data|
      if data.value.present?
        puts "#{data.prompt} (#{data.value}):"
      else
        puts "#{data.prompt}:"
      end
      template.set(data, gets.chomp)
    end
    puts

    @collection = client.add_template(collection, template)
  end

  def delete
    item = choose_item
    @collection = client.delete(item)
  end

  def edit
    item = choose_item

    @collection = client.get(item.href)

    item = collection.items.first
    template = collection.template

    item.data.each do |data|
      next unless template.data.any? { |td| td.name == data.name }
      if data.value.present?
        puts "#{data.prompt} (#{data.value}):"
      else
        puts "#{data.prompt}:"
      end
      template.set(data, gets.chomp)
    end

    @collection = client.update(item, template)
  end

  private

  def client
    @client ||= Client.new("http://hyper-hackery.herokuapp.com/")
  end

  def choose_item
    print_items(collection.items, true)

    puts "Choose an item"
    item_number = gets.chomp.to_i

    collection.items[item_number]
  end

  def print_items(items, include_numbers = false)
    puts "Returned items"
    items.each_with_index do |item, index|
      value = item.attribute("completed").value
      finished = value == "true" || value == true ? "(X)" : "( )"
      number = " (#{index})" if include_numbers
      puts "  - #{finished}#{number} #{item.attribute("title").value}"
    end
    puts
  end
end

CommandLine.new.run
```

Here is a sample run:

```
ruby perform_search.rb
What do you want to do?
  - Refresh Items (refresh)
  - Items (items)
  - Queries (queries)
  - Template 'Add ToDo' (template)
  - Delete (delete)
  - Edit (edit)
  - Exit (exit)
items

Returned items
  - (X) one more test again
  - ( ) danny boy
  - ( ) fishing
  - ( ) goofing around
  - ( ) one more simple test
  - (X) one more minor test
  - ( ) update these docs
  - ( ) wheee
  - ( ) asdasd
  - ( ) something to test
  - (X) update cj parser
  - ( ) trying to test
  - ( ) one more simple test
  - (X) additional stuff
  - ( ) ssssss
  - (X) more stuff
  - ( ) one more simple test

What do you want to do?
  - Refresh Items (refresh)
  - Items (items)
  - Queries (queries)
  - Template 'Add ToDo' (template)
  - Delete (delete)
  - Edit (edit)
  - Exit (exit)
queries

Active ToDos (active collection)
Completed ToDos (completed collection)
Search ToDos (search)
Choose a query
search

Title:
one

Returned items
  - (X) one more test again
  - ( ) one more simple test
  - (X) one more minor test
  - ( ) one more simple test
  - ( ) one more simple test

What do you want to do?
  - Refresh Items (refresh)
  - Items (items)
  - Queries (queries)
  - Template 'Add ToDo' (template)
  - Delete (delete)
  - Edit (edit)
  - Exit (exit)
exit
```

[restfest]: http://www.restfest.org/
[cj-source]: https://github.com/RESTFest/2015-Greenville/tree/master/collection-json-ruby-client
[collectionjson]: http://amundsen.com/media-types/collection/
