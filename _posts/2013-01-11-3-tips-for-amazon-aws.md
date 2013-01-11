---
layout: post
categories:
- aws
- ec2
title: 3 Tips for Amazon AWS
---

I set up several EC2 instances earlier this week and had a lot of trouble trying to get them up and running. I figured I would share the things I learned, even if they seem pretty obvious.


#### When setting a specific kernel and ram id in AWS, the architecture should be chosen first.

I had to create an AMI from a specific snapshot and to do that you need to set the correct kernel and ram id. I could not find the one that I was supposed to use and eventually switched the architecture from 32 to 64, and suddenly the ids I was looking for were there.


#### If your instance is not responding, you might have picked the wrong kernel.

Since I was picking my own kernel id from a pretty huge select box, I managed to get it wrong. The instance launched and the AWS console said it couldn't reach it via network. Loading it in the browser was broken as well. I tried launching new instances and rebooting several times to no avail. Eventually I checked the ids and they were off. Creating a new AMI with the correct kernel id fixed it.


#### If you need to transfer a file between instances, transfer between them directly.

This one might seem incredibly obvious, but for some reason I didn't think to do this until after I was half way through downloading a 3GB+ file. Once I realized it, I started the transfer between them directly and the transfer between them finished before my download did.
