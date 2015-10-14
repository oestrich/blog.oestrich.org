---
layout: post
categories:
- api
- rails
title: Turning Controller Actions into Services
---

_This post was originally published on the_
[SmartLogic Blog](http://blog.smartlogicsolutions.com/2012/10/25/api-development/).

<img class="float-img" title="SRP keeps your code from behaving like this. " src="/images/bigonemanband.jpg" alt="SRP keeps your code from behaving like this. " width="213" height="320" />

When you're developing an API, controller actions have a tendency to get large and inefficient. You can keep chopping controller actions into smaller pieces, but turning them into services may be where you want to go. The main reason for this is because you can get validation of the data before you use it to perform the service, making code more reliable.

### What is a service and when would you need to make one?

A service is a class that interacts with multiple models. If a method on a model is going to access more than itself and maybe one neighbor then it should be pulled out into a service. [Single Responsibility Principle (SRP)](http://en.wikipedia.org/wiki/Single_responsibility_principle) essentially states that a class should do just one thing. With this, we're making our class do just one thing: create an order. That way, we don't have a "fat model" that violates SRP.

Here's an example app on [github](http://github.com/oestrich/services_example) to supplement the code below.

##### app/controllers/orders_controller.rb
```ruby
class OrdersController < ApplicationController
  def create
    service = OrderCreationService.new(current_user, params[:order])
    service.perform

    if service.successful?
      respond_with service.order
    else
      respond_with service.errors, :status => 422
    end
  end
end
```

##### app/services/order_creation_service.rb
```ruby
class OrderCreationService
  include ActiveModel::Validations

  validates :name, :presence => true

  attr_reader :user, :params, :order, :name

  delegate :as_json, :to => :order

  def initialize(user, params)
    @user = user
    @params = params

    params.each do |param, value|
      instance_variable_set("@#{param}", value) if respond_to?(param)
    end
  end

  def perform
    return unless valid?

    # This can be done with only one line, or you can go nuts and make
    # this much more expansive if your desired functionality demands it
    @order = user.orders.create(params)
  end

  def successful?
    valid? && order.persisted?
  end
end
```

Here's how this gives you the nice ability of having validations around the parameters your API takes:

```bash
$ curl -X POST http://localhost:3000/orders
{ 'errors': { 'name': ["can't be blank"] } }
```

With this method, your <a href="http://blog.smartlogicsolutions.com/2012/08/28/api-planning-and-proceeding-tell-me-what-youre-working-with/" target="_blank">API development</a> results in cleaner, more reliable code that will be easier for both internal developers and external developers to work with. Also, be sure to test via <a href="http://blog.smartlogicsolutions.com/2012/07/12/curlin-for-docs/" target="_blank">RspecApiDocumentation and Raddocs</a>.

<em>For more like this, follow SmartLogic <a href="http://www.linkedin.com/company/smartlogic-solutions" target="blank_">on LinkedIn</a> or <a href="http://facebook.com/smartlogic" target="blank_">like us on Facebook.</a></em>
<br>
<a href="http://www.flickr.com/photos/megangoodchild/4675235489/" target="_blank">Image Source</a>
