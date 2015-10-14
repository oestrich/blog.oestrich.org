---
layout: post
categories:
- api
- rails
title: Using let in Controllers
---

`let` in rspec is one of my favorite features. The [decent_exposure](https://github.com/voxdolo/decent_exposure) is similar to `let`, but it ends up doing way more for you than I want it to. I have been doing mostly API work in Rails recently so it doesn't fit.

To fix this I ended up writing my own simple version of `let` to use in my controllers.

##### lib/let.rb
```ruby
def self.let(method, &block)
  define_method(method) do
    @lets ||= {}
    @lets[method] ||= instance_eval(&block)
  end
end
```

Since I ended up pasting this code in most of the projects I'm in, I ended up writing a gem [letter](https://github.com/oestrich/letter). It's simple to use.

##### app/controllers/application_controller.rb
```ruby
class ApplicationController
  extend Letter

  let(:current_user) { User.find(session[:user_id]) }
end
```

Currently this gem doesn't work if you have views. So a simple addition to make it work with views would be:

##### app/controllers/application_controller.rb
```ruby
class ApplicationController
  extend Letter

  def self.let(method, *args)
    super
    helper_method(method)
  end

  let(:current_user) { User.find(session[:user_id]) }
end
```
