---
layout: post
categories:
- blog
title: Jekyll Git Deployment With Bundler
---

I recently converted my blog over to Jekyll, mainly to not have MySQL running. One thing I did though was set up git deployment. I mostly followed the Jekyll guide, but I different when setting up the jekyll gem.

The starting post-receive from [Jekyll Deployment Guide](https://github.com/mojombo/jekyll/wiki/Deployment).
##### hooks/post-receive (original)
{% highlight bash %}
GIT_REPO=$HOME/myrepo.git
TMP_GIT_CLONE=$HOME/tmp/myrepo
PUBLIC_WWW=/var/www/myrepo

git clone $GIT_REPO $TMP_GIT_CLONE
jekyll --no-auto $TMP_GIT_CLONE $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit
{% endhighlight %}

I did `bundler install --deployment` so that it would be in the local directory. The problem with this is that jekyll then picks up the test posts in the gem as your posts. Now I had a ton of bad posts on my blog. (You may have noticed them in the feed.)

I also set it up so that it doesn't blow away the repo (and the gems) every time I push up a new post.

##### hooks/post-receive (final)
{% highlight bash %}
GIT_REPO=$HOME/repos/blog.oestrich.org
TMP_GIT_CLONE=$HOME/tmp/blog.oestrich.org
PUBLIC_WWW=$HOME/vhosts/blog.oestrich.org

unset GIT_DIR

if [ -d $TMP_GIT_CLONE ]; then
  cd $TMP_GIT_CLONE

  git pull origin master
else
  git clone $GIT_REPO $TMP_GIT_CLONE

  cd $TMP_GIT_CLONE
fi

bundle install --deployment

# Remove the sample jekyll site from the test suite
rm -rf $TMP_GIT_CLONE/vendor/bundle/ruby/1.9.1/gems/jekyll-0.11.2/test/source

bundle exec jekyll --no-auto $TMP_GIT_CLONE $PUBLIC_WWW

exit
{% endhighlight %}
