---
layout: post
categories:
- Linux
- Ruby
title: RVM and Cron
---

Today I set up my first real rails production server and in the process ran across an issue with cron and rvm. I have a rake task that I want to run every 15 minutes. I made a script that cd'd into the directory, loaded the correct ruby and gemset then did the rake task.

Every time cron ran though, it kept giving me the following error:

```bash
rvm: command not found
rake: command not found
```

After some quick googling I came across two posts that let me do the rake task:

*  [http://blog.scoutapp.com/articles/2010/09/07/rvm-and-cron-in-production](http://blog.scoutapp.com/articles/2010/09/07/rvm-and-cron-in-production)
*  [http://groups.google.com/group/rubyversionmanager/msg/9c9ddcb79beafe13](http://groups.google.com/group/rubyversionmanager/msg/9c9ddcb79beafe13)

Sourcing the rvm script was the part I was missing.

This was the final script I came up with:

```bash
#!/bin/bash

source ~/.rvm/scripts/rvm
cd /home/deploy/apps/my_app
rvm use 1.9.2@my_app
RAILS_ENV=production rake email:reminder
```
