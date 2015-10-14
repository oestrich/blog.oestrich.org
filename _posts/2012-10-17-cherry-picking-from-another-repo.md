---
layout: post
categories:
- git
title: cherry-pick'ing From Another Repo
---

<img src="/images/cherry-picking.jpeg" alt="Cherry picking" class="float-img" title="Photo by InGale2005" />

We're in the process of splitting out a fairly large rails app that contains very clear "mini" apps. Each "mini" app will get their own full app and of course their own git repo.

As we split out the multiple apps, some commits were bound to be missed because others continued working in the main repo. After doing a bit of searching I found that there was a nice way to import these commits via `cherry-pick`.

<br class="clear" />
```bash
git remote add other-repo ../other-repo/
git fetch other-repo
git cherry-pick cf0dbfe3c63ab0f49bb8fa97d3b28bb0bd9eb896
```

I've ended up using this trick several times this week and it's turned out to be an excellent time saver.

##### Resources
1. [StackOverflow](http://stackoverflow.com/questions/5120038/is-it-possible-to-cherry-pick-a-commit-from-another-git-repository)
1. [git-cherry-pick](http://www.kernel.org/pub/software/scm/git/docs/git-cherry-pick.html)
1. [Photo by InGale2005](http://www.flickr.com/photos/igal/7901479448/in/photostream/)
