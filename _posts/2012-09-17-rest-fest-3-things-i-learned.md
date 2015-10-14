---
layout: post
categories:
- restfest
- apis
title: REST Fest - 3 Things I Learned
---

_This post was originally published on the_
[SmartLogic Blog](http://blog.smartlogicsolutions.com/2012/09/17/rest-fest-3-things-i-learned/).

This past weekend, I headed down to REST Fest in Greenville, South Carolina. My weekend lived up to [my expectations](http://blog.smartlogicsolutions.com/2012/09/11/rest-fest-3-things-im-looking-forward-to): though I didn't get much of a chance to work on [Frenetic](https://github.com/dlindahl/frenetic), I spread the word about [Raddocs and RAD](http://blog.smartlogicsolutions.com/2012/07/12/curlin-for-docs/), and learned a lot from the API developers at the event.

For those of you who couldn't make it, here's a quick wrap-up of three things I learned, and, at the end of the post, the video of my 5in5 talk:

### What a bad API looks like
During [Mike Amundsen](https://twitter.com/mamund)'s [presentation](http://vimeo.com/channels/restfest/49503453) he said, "Exposing your database is like showing me your underwear. I don't wanna see that." This was both enlightening and disheartening. My previous APIs have been mostly database table to resource. While they do their job, they definitely could be better. In the future I will definitely put more thought into the design of my APIs.

### Registering Link Relations and Media Types
Before attending REST Fest, I didn't know that if you create a short link relation you have to register it with a registry such as [IANA](http://www.iana.org/assignments/link-relations/link-relations.xml) or [MicroFormats](http://microformats.org/wiki/rel-values). The way around this is to create link relations as URIs.

An example from the Helpdesk Hackday at REST Fest:

```xml
<atom:link rel="http://helpdesk.hackday.2012.restfest.org/rels/ticket" href="http://.../tickets/9172361" type="application/vnd.org.restfest.2012.hackday+xml" />
```

In the same vein as registering link relations, I didn't know you were supposed to register media types. The HTTP spec calls for them to be registered in [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.7). Here's the gist: "Media-type values are registered with the Internet Assigned Number Authority (IANA [19]). The media type registration process is outlined in RFC 1590 [17]. Use of non-registered media types is discouraged."

### Linking in JSON
I learned of a new hypermedia format for JSON called Siren. I had previously seen HAL before, but I like that Siren gives you the ability to include actions in the resource. Siren is located [here on GitHub](https://github.com/kevinswiber/siren).

In addition to learning about Siren, I also learned more about Collection+JSON. I had seen it before but I had only glanced at it. Collection+JSON is located [here](http://amundsen.com/media-types/collection/format/). I will definitely be giving both of these a more thorough look through for my next API project.

Overall, the weekend was jam packed with new information. Were you there? What was the most useful thing you learned? Comment here or tweet me, [@ericoestrich](https://twitter.com/ericoestrich).

And, as promised, here's my 5in5 talk:
<iframe src="http://player.vimeo.com/video/49504102" frameborder="0" width="500" height="281"></iframe>

<a href="http://vimeo.com/49504102" target="_blank">REST Fest 2012 \ Eric Oestrich \ FiveInFive</a> from <a href="http://vimeo.com/restfest" target="_blank">REST Fest</a> on <a href="http://vimeo.com" target="_blank">Vimeo</a>.
