---
layout: post
categories:
- raspberry pi
- linux
title: Updating the Bramble
---

Having based my Raspberry Pi cluster on Arch Linux, I need to update the bramble fairly regularly. I have a weekly event on Sundays that reminds me to keep things up to date.

My updating process:

- Update all docker hosts
- Update all file system hosts
- Update the rest

Normally this is a very uneventful process as nothing breaks. Just in case something does though, I always try to have a "canary" raspberry pi that gets updated first and doesn't have anything important on it. The last time I updated something did go wrong and I was very glad of this process.

## Something went wrong

When I updated my canary pi nothing went wrong. Everything looked good. I continued on to update pis that had slightly more important things on it. Eventually I got to a pi that had a docker container on it.

After restarting to complete the update (the kernel was updated), I checked to see that all of the containers started. When they didn't I noticed I couldn't run any containers with a failure of "socket operation on non-socket".

I found [this docker issue on github][githubissue] and was able to downgrade to the previous docker I had installed. Everything was working again and I didn't take down anything important.

Docker has since updated to 1.7.1 (not yet released, very soon) and fixed the issue.

## Conclusion

Always be careful when running Arch as a server OS and make sure to have some sort of early warning system with updates.

[githubissue]: https://github.com/docker/docker/issues/14184
