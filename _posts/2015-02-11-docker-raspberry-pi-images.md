---
layout: post
categories:
- docker
- raspberry pi
title: Docker Raspberry Pi Images
---

For a while I've been interested in doing some kind of cluster with Raspberry Pis. When the second version came out, I decided to look into docker based deployment. I found some really cool stuff in CoreOS, but I doubt we'll get that any time soon. For now I will make due with a plain Arch install running docker.

Docker was easy to install on Arch, but I ran into a problem fairly quickly. I couldn't find any images that were built for the ARM platform on the docker hub. This seemed like a good chance to try out building a docker image from scratch.

##### oestrich/arch-pi Dockerfile
{% highlight docker %}
FROM scratch
COPY . /
{% endhighlight %}

The `arch-pi` image is the latest raspberry pi Arch tar.gz that is extracted into the same folder as the `Dockerfile`. I use `arch-chroot` on that folder to be able to remove a few packages first. This helps shrink the image considerably. I removed most of the firmware and kernels. Since docker uses the host kernel, this is OK to do.


##### oestrich/base-pi Dockerfile
{% highlight docker %}
FROM oestrich/arch-pi
MAINTAINER Eric Oestrich <eric@oestrich.org>

RUN pacman -Syu --noconfirm
RUN pacman -S --noconfirm \
      base-devel \
      libffi \
      libyaml \
      openssl \
      zlib \
      git

RUN git clone https://github.com/sstephenson/ruby-build.git
RUN cd ruby-build && ./install.sh
RUN ruby-build 2.2.0 /opt/ruby
ENV PATH $PATH:/opt/ruby/bin
{% endhighlight %}

This docker image installs everything needed to get ruby installed. Note that this will take a good hour or so to build. I did discover that the Raspberry Pi was quicker than the old t1.micro instance on AWS, go Raspberry Pi.


##### oestrich/base-pi-web Dockerfile
{% highlight docker %}
FROM oestrich/base-pi
MAINTAINER Eric Oestrich <eric@oestrich.org>

RUN pacman -Syu --noconfirm
RUN pacman -S --noconfirm \
      libxml2 \
      libxslt \
      imagemagick \
      openssl \
      postgresql \
      python2

RUN gem install bundler
RUN gem install foreman
RUN gem install libv8 --version 3.16.14.7
RUN gem install nokogiri --version 1.6.5
{% endhighlight %}

This docker image gets the base environment set for a rails app. Whatever system libraries are required and some base gems like `bundler` and `foreman`. I also installed `libv8` and `nokogiri` because together they take about 40 minutes to install. I used the `--version` flag to install the exact version that bundler will install. This lets bundler simple use the installed gem, saving 40 minutes each time the Gemfile changes.

##### project/web Dockerfile
{% highlight docker %}
FROM oestrich/base-pi-web
MAINTAINER Eric Oestrich <eric@oestrich.org>

RUN mkdir -p /apps/project

ADD Gemfile* /apps/project/

WORKDIR /apps/project
RUN bundle -j4 --without development test
ADD . /apps/project
ADD .env.production /apps/project/.env

RUN . /apps/project/.env && bundle exec rake assets:precompile

CMD ["foreman", "start", "web"]
{% endhighlight %}

This last docker image installs a rails app. It copies over the `Gemfile` and `Gemfile.lock` into the container first to install gems. This way you only rebundle if your `Gemfile` changes, saving time on each deploy. I also want to point out the `-j4` option on the bundle. The Raspberry Pi 2 is quad core, so let's use all the power available. Assets are compiled and then ends with a `CMD` of `foreman start web`.

An alternative to `CMD` is to use `ENTRYPOINT`:
{% highlight docker %}
ENTRYPOINT ["foreman", "start"]
{% endhighlight %}

This will let you use different foreman apps without having to build a separate image for each one.

{% highlight bash %}
docker run -t -i -p 5000:5000 project/web worker
{% endhighlight %}
