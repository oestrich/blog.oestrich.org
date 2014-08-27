---
layout: post
categories:
- conference
- restfest
title: REST Fest - 3 Things I'm Looking Forward To
---

_This post was originally published on the_
[SmartLogic Blog](http://blog.smartlogicsolutions.com/2012/09/11/rest-fest-3-things-im-looking-forward-to/).

<img style="float: right; margin-left: 10px;" title="Rest Fest 2012" src="/images/restfest.png" alt="Rest Fest 2012" width="200" height="200" />

I am pretty excited about attending [REST Fest](http://www.restfest.org/) this week. I am most looking forward to the following:

#### Getting together with other API developers

Being able to hang out with other developers who are interested in [APIs](http://blog.smartlogicsolutions.com/2012/08/28/api-planning-and-proceeding-tell-me-what-youre-working-with/) is definitely the thing I'm most looking forward to. I want to absorb as much information as I can from this event.

#### Hacking on Frenetic

On the Thursday Hack day I want to play around with the gem Frenetic, [https://github.com/dlindahl/frenetic](https://github.com/dlindahl/frenetic). Right now it's kinda awkward to use it as a client because it requires a crazy amount of chaining. I would like to be able to knock that down a lot.

{% highlight ruby %}
api.get(api.description.links.orders.href).body.resources.orders
{% endhighlight %}

In addition to getting the previous command shorter, Frenetic doesn't seem to give you easy accessors to the links that the API returns.

#### Talking more about Raddocs and RAD

Since I'm going to be giving a 5in5 talk when I attend REST Fest, it's the perfect opportunity to put [Raddocs and RAD](http://blog.smartlogicsolutions.com/2012/07/12/curlin-for-docs/) in front of other developers faces. I will be giving a modified version of my ["cURLin' for Docs"](http://www.slideshare.net/SmartLogic/curlin-for-docs) presentation.

Looking forward to heading to Greenville! Tweet me [@ericoestrich](https://twitter.com/ericoestrich) if you want to meet up when we're there.
