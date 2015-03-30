---
layout: post
categories:
- docker
- git
- rails
title: Git push with docker on a Raspberry Pi 'Bramble'
---

For my [Raspberry Pi bramble]({% post_url 2015-03-17-nginx-docker-container %}), I have a build server that builds all of my docker containers and then deploys them. This is how I have that set up.

## Folder Structure

On the build server I have a deploy user that has the following folder structure:

```
~/apps/
  project/
    repo/
    build/
    config/
```

The `repo` folder contains a bare git repository that git clients will push to similar to Heroku and Github. The build folder is a clone of the `repo` with a working copy for docker to use as a build context. Lastly the `config` folder contains any configuration that will be copied into the docker container as necessary, such as a `.env` file.


## post-receive hook

When a git client pushes to the server a `post-receive` hook is called. This is the script. The script is located at `repo/hooks/post-receive` and needs to be executable by the deploy user.

``` bash
APP_PATH=/home/deploy/apps/project
DOCKER_TAG="registry.example.com/project/web"

REPO_PATH=$APP_PATH/repo
BUILD_PATH=$APP_PATH/build
CONFIG_PATH=$APP_PATH/config

unset GIT_DIR
cd $BUILD_PATH
git fetch
git reset --hard origin/master

cp $CONFIG_PATH/.env $BUILD_PATH/.env

docker build -t $DOCKER_TAG .
docker push $DOCKER_TAG

echo 'Restarting on docker-host'
# Double quotes are necessary for the next line to interpolate
# the $DOCKER_TAG variable
ssh deploy@docker-host.example.com "docker pull $DOCKER_TAG"
ssh deploy@docker-host.example.com "sudo systemctl restart project"
```

The first half of the script sets up variables and should be self explanatory. Unsetting `GIT_DIR` is required because otherwise git will think you are still in the bare repo and complain about not having a working directory. The repository is reset completely to whatever is now master and then build with docker.

The docker tag is important because it needs to be named however you access your docker repository. Mine for instance is at `reigstery.example.com`.

Once built and pushed to the docker registry, I pull the new docker image on the remote docker host and restart the project via systemd.

## Project docker file

For the project docker file [see my post on running a rails app in docker]({% post_url 2015-02-11-docker-raspberry-pi-images %}) for a full explanation. Below is just the project `Dockerfile`.

``` docker
FROM oestrich/base-pi-web
MAINTAINER Eric Oestrich <eric@oestrich.org>

RUN mkdir -p /apps/project

ADD Gemfile* /apps/project/

WORKDIR /apps/project
RUN bundle -j4 --without development test
ADD . /apps/project

RUN . /apps/project/.env && bundle exec rake assets:precompile
RUN . /apps/project/.env && bundle exec rake db:migrate

ENTRYPOINT ["foreman", "start"]
```

The `Gemfile` is added first to allow docker to skip reinstalling gems if it never changes. This is incredibly useful as it takes a very long time to build on the Raspberry Pi.

Asset precompilation comes next along with the database migration. I include a database migration each build because I don't have a nice way to access the database server with a production rails app right now.

The entrypoint is set to `foreman start` and the app will be launched with a `CMD` of `web` or `worker`.

## systemd service file

This is my systemd service that is installed on the docker hosts to keep the rails app alive.

```
[Unit]
Description=Project Web
Requires=docker.service
After=docker.service

[Service]
ExecStart=/usr/bin/docker run --rm -p 5000:5000 --name project-web registry.example.com:5000/project/web web
Restart=always

[Install]
WantedBy=multi-user.target
```

This file is very simple and keeps the docker container alive no matter what.

## Future improvements

While this works very well for what I want (a small cluster in my house), there are a number of improvements I'd like to eventually get to. I'd really like to be able to use [CoreOS][coreos] and all of the cool tools it comes with. I can't at the moment because CoreOS does not support the ARM platform.

Some of the neat tools include `etcd` and `fleet` to allow me to more easily deploy containers to the bramble without having to know exactly what host it will be on like above. This is the biggest improvement I would like to get.

I'd also like to be able to cache gem building for rails apps. Building `nokogiri` on a Raspberry Pi takes about half an hour. Since I have no caching in place, any time the `Gemfile` changes it requires a complete rebuild of all gems. I think I can fix this by using [glusterfs][glusterfs] (which I already have in place for uploads.) This might be a little slow, but I'm hoping it will still be faster than a complete rebuild. There is a similar problem with asset compilation.

In future blog posts I hope to address these problems with what I have found.

[coreos]: http://coreos.com
[glusterfs]: http://www.gluster.org/
