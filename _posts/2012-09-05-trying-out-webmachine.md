---
layout: post
categories:
- ruby
- webmachine
title: Trying out WebMachine
---

Yesterday I tried out [webmachine-ruby](https://github.com/seancribbs/webmachine-ruby). Overall it left a good impression on me. One thing I really liked was the separation of a single resource and a collection resource.

{% highlight ruby %}
App = Webmachine::Application.new do |app|
  app.routes do
    add ["orders"], OrdersResource
    add ["orders", :id], OrderResource
  end
end
{% endhighlight %}

Any call to "/orders" will go to the OrdersResource, since you're acting on a collection, and any call with an "id" in it will go to the single resource.

I also really liked the idea of callbacks on a resource. To point out one, in order to get a resource to respond to a certain method, you add it to the `#allowed_methods`:

{% highlight ruby %}
class OrderResource 
  def allowed_methods
    ["GET", "POST"]
  end

  def content_types_provided
    [["application/json", :to_json]]
  end
end
{% endhighlight %}

The `#content_types_provided` callback will call `#to_json` if it sees "application/json".

One pretty big issue I faced was a lack of examples on how to use its statemachine. The example apps listed only do `GET` and `POST` requests but there wasn't an example on how do `DELETE` or `PUT` something. Luckily I had access to [someone who has used it before](https://github.com/samwgoldman) and could help me with the issues I came across. I'm hoping this example app can be of use to someone else looking into WebMachine.

My resulting example app is [here](https://gist.github.com/3638605) as a gist and I've embedded the main app file.

{% highlight ruby %}
require 'webmachine'
require 'webmachine/adapters/rack'
require 'json'

class Order
  attr_accessor :id, :email, :date

  DB = {}

  def self.all
    DB.values
  end

  def self.find(id)
    DB[id]
  end

  def self.next_id
    DB.keys.max.to_i + 1
  end

  def self.delete_all
    DB.clear
  end

  def to_json(options = {})
    %{{"email":"#@email", "date":"#@date", "id":#@id}}
  end

  def initialize(attrs = {})
    attrs.each do |attr, value|
      send("#{attr}=", value) if respond_to?(attr)
    end
  end

  def save(id = nil)
    self.id = id || self.class.next_id
    DB[self.id] = self
  end

  def destroy
    DB.delete(id)
  end
end

class JsonResource < Webmachine::Resource
  def content_types_provided
    [["application/json", :to_json]]
  end

  def content_types_accepted
    [["application/json", :from_json]]
  end

  private
  def params
    JSON.parse(request.body.to_s)
  end
end

class OrdersResource < JsonResource
  def allowed_methods
    ["GET", "POST"]
  end

  def to_json
    {
      :orders => Order.all
    }.to_json
  end

  def create_path
    @id = Order.next_id
    "/orders/#@id"
  end

  def post_is_create?
    true
  end

  private
  def from_json
    order = Order.new(params).save(@id)
  end
end

class OrderResource < JsonResource
  def allowed_methods
    ["GET", "DELETE", "PUT"]
  end

  def id
    request.path_info[:id].to_i
  end

  def delete_resource
    Order.find(id).destroy
    true
  end

  def to_json
    order = Order.find(id)
    order.to_json
  end

  private
  def from_json
    order = Order.new(params)
    order.save(id)
  end
end

App = Webmachine::Application.new do |app|
  app.routes do
    add ["orders"], OrdersResource
    add ["orders", :id], OrderResource
    add ['trace', '*'], Webmachine::Trace::TraceResource
  end
end
{% endhighlight %}
