---
layout: post
categories:
- Rails
title: Devise Authentication Token
---

I started using the devise authentication token for a project recently and was confounded on how to get the token from the client's perspective. After fiddling with different ways, I finally found that adding :authentication_token to attr_accessible let's it slide through the post to sign_in.

{% highlight ruby %}
class User
  attr_accessible :authentication_token
end
{% endhighlight %}

<br />

{% highlight bash %}
    curl -d "user[email]=eric@example.com&amp;user[password]=password" 
    http://example.com/users/sign_in.json
{% endhighlight %}

<br />

{% highlight json %}
    {"authentication_token":"RxYA7tzybe8E3H4q1zvb",
    "email":"eric@example.com"}
{% endhighlight %}
