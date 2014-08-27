---
layout: post
categories:
- ruby
- rails
- hypermedia
- hal
title: Abusing ActiveModel::Serializers for HAL
---

For my recent APIs I've used JSON HAL as the media type and ActiveModel::Serializers as an alternative to `#as_json`. It's worked out fairly well except I've had to bend backwards in order to get ActiveModel::Serializers to spit out HAL.

Below are some common things I've had to use. You can check out an example [here](https://github.com/oestrich/hypermedia_rails).

### Disable Root
This one is pretty easy, we just need to not have the element type be the root key.

{% highlight ruby %}
class OrderSerializer < ActiveModel::Serializer
  root false
end
{% endhighlight %}

### Embedded Resources
Since HAL resources have other resources in the "\_embedded" key, we need to make sure collections show up there.

{% highlight ruby %}
class OrderSerializer < ActiveModel::Serializer
  has_many :items

  def as_json(*args)
    hash = super

    hash[:_embedded] = { :items => hash.delete(:items) }

    hash
  end
end
{% endhighlight %}

### Links
Adding links is pretty easy with ActiveModel::Serializers, just add a new attribute. One thing to watch out for, the collection serializers don't have access to url helpers.

{% highlight ruby %}
class OrderSerializer < ActiveModel::Serializer
  attributes :_links

  def _links
    {
      :self => {
        :href => orders_url(order)
      }
    }
  end
end
{% endhighlight %}

### Slimming Down Serializers
I wanted to use the same serializer for both an index and a show resource, but they didn't show exactly the same amount of information. I fixed this by sending in an extra option on the show page: `respond_with order, :expand => true`.

{% highlight ruby %}
class OrderSerializer < ActiveModel::Serializer
  attributes :email, :extra_info

  has_many :items

  def include_extra_info?
    @options[:expand]
  end

  def include_associations!
    if @options[:expand]
      include! :items
    end
  end
end
{% endhighlight %}
