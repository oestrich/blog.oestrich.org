---
layout: default
categories:
- Rails
title: Devise Authentication Token
---

# [{{ page.title}}]({{ page.url }})
<span>Posted on {{ page.date | date_to_string }}</span>

I started using the devise authentication token for a project recently and was confounded on how to get the token from the client's perspective. After fiddling with different ways, I finally found that adding :authentication_token to attr_accessible let's it slide through the post to sign_in.

    class User
      attr_accessible :authentication_token
    end

<br />

    curl -d "user[email]=eric@example.com&amp;user[password]=password" 
    http://example.com/users/sign_in.json

<br />

    {"authentication_token":"RxYA7tzybe8E3H4q1zvb",
    "email":"eric@example.com"}
