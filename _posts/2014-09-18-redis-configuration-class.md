---
layout: post
categories:
- redis
- ruby
- config
title: Redis Application Configuration Class
---

At work I needed to make a configuration class that worked well with ActiveAdmin, which meant implementing a lot of the ActiveModel modules. Below is what I came up with. It was pretty overkill for this project, but I like what came out nonetheless.

One nice thing I want to point out is that I'm passing in a redis instance, in this case the default is the AppContainer.redis. This lets you easily set up a `Redis::Namespace` for the configuration class, and the class itself doesn't have to know how to do that.

To define new attributes you use the `attribute` class method. It lets you set a default adn a type. The type is only special for `:integer` at the moment, but it's not hard to add more.

```ruby
class Configuration
  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Dirty
  include ActiveModel::Serialization
  include ActiveModel::Serializers::JSON

  def self.reset!(redis = AppContainer.redis)
    attributes.each do |attribute|
      redis.del("config_#{attribute}")
    end
  end

  def self.attributes
    @attributes ||= []
  end

  def self.attribute(name, default: default, type: type)
    attributes << name

    define_attribute_methods name

    define_method(name) do
      attr = @attributes[name] || default

      case type
      when :integer
        attr.to_i
      else
        attr
      end
    end
    define_method("#{name}=") do |value|
      send("#{name}_will_change!") unless value == @attributes[name]
      @attributes[name] = value
    end
  end

  attribute :timeout, default: 1500, type: :integer

  def initialize(redis = AppContainer.redis)
    @redis = redis
    @attributes = {}
    load_attributes
  end

  def attributes
    attrs = self.class.attributes.map do |attribute|
      [attribute, send(attribute)]
    end
    Hash[attrs]
  end

  def update(attributes)
    attributes.each do |attribute, value|
      send("#{attribute}=", value)
    end
    save
  end

  def save
    changes.each do |attribute, (old, new)|
      @redis.set("config_#{attribute}", new)
    end
    changes_applied
    true
  end

  private

  def load_attributes
    self.class.attributes.each do |attribute|
      @attributes[attribute] = @redis.get("config_#{attribute}")
    end
  end
end
```

To use it:

```ruby
config = Configuration.new
config.timeout
# => 1500
config.timeout = 1600
config.changed?
# => true
config.changes
# => { "timeout" => [1500, 1600] }
config = Configuration.new
config.timeout
# => 1600
config.update("timeout" => 1000)
config.timeout
# => 1000
```
