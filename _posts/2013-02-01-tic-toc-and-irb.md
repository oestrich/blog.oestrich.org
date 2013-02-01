---
layout: post
categories:
- ruby
title: Tic Toc And IRB
---

My co-worker, [Sam Goldman](http://twitter.com/nontrivialzeros), a while back showed me this useful little IRB trick in order to get out how long a method takes. I used it pretty heavily recently with doing a call that hits the network.

It's really easy to set up, just add the following to your `.irbrc` file.

##### ~/.irbrc

    def tic
      @tic = Time.now
    end

    def toc
      "#{Time.now - @tic} seconds"
    end

Once that's set up you can call it surrouding the method you want to time.

    irb(main):003:0> tic; puts "Hello"; toc
    Hello
    => "3.0686e-05 seconds"
    irb(main):004:0> tic; sleep 2; toc
    => "2.000082375 seconds"

It's also in my dotfiles repo on [github](https://github.com/oestrich/dotfiles/blob/master/irbrc).
