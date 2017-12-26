---
layout: post
categories:
- elixir
- ex_venture
title: ExVenture Updates for December 2017
description: ExVenture changes in the month of December 2017
date: 2017-12-26 9:00AM
---

The last month of [ExVenture][exventure-github] mostly saw behind the scenes updates. Lots of small tweaks here and there in the admin interface along with documentation of modules and functions.

The documentation website is [exventure.org][exventure]. You can see the latest additions here on [MidMUD][midmud], my running instance of ExVenture.

## Effects updates

Items that you are wielding are now included in the effects calculation. So if you use a dagger to use a magic skill, the dagger's damage will also be included, but the magic damage can be halved due to a `damage/type` restriction on the dagger. So wielding the correct weapons will help maximize your damage output.

Also new are continuous damage effects, `damage/over-time`. These deal immediate damage and then tick `every` milliseconds, up to a maximum `count`. This can be used for things like poison.

```
{
  'kind': 'damage/over-time',
  'type': 'slashing',
  'amount': 10,
  'every': 1000,
  'count': 4
}
```

Finally there is a `recover` effect that `healing` was converted to. This is a more generic version that can recover skill points and move points along with health points.

## Pagination and filtering

A lot of the admin sections now paginate and have a filter on the side for searching. This started to become required as I filled in more things in [MidMUD][midmud]. Here is the NPC admin:

![NPC Admin Filtering](https://exventure.org/images/admin-npc-index.png)

## User admin

The User admin got some sprucing up, the index has pagination and filtering now along with displaying more information. The show for a user displays about the same information but does it in a lot nicer way. When someone is signed in you can also see live stats such as which room they are in.

![User Admin Index](/images/exventure-2017-12-users.png)

![User Admin Show](/images/exventure-2017-12-user.png)

## Item Admin

I've recently put a lot of work into the item admin. I want to have a more filled in world as people start stumbling across the game. The items index was a simple list before adding fitlering to it. I also renamed `ItemTag` to `ItemAspect` to allow a normal `tags` field to sit on items. Item cloning was also added (pre-fill the new form) to help speed up sets of items, such as a full leather armor set.

![Item Admin Index](/images/exventure-2017-12-items.png)

## Use items

A new command was added `use` which uses an item in your inventory on yourself. There is no ability to target yet, it simply applies the effects to yourself, good or bad. That means you can `use sword` on yourself and take damage.

## Shop command interface

I log commands that people try to use and get a parse error. A _lot_ of these were related to the shops command. I tried updating that to make it more usable. Most interfacing with shops will now be a lot more smooth when there is only one shop is the room. I also added a final parse catch to display help information if the command starts with `shop`. Hopefully there will be less troubles with this part of the game in the future.

## Small tweaks

- Game inventory collapses duplicate items
- Edit game name and MOTD from the web panel
- Slightly nicer prompt output for dealing continuous effects/using effects

## Social Updates

I posted about someone finding the game and playing around for 15+ minutes on twitter, and included a photo of their stats. That was retweeted by [@elixirlang][elixirlang] and a lot of people found the project because of it. Welcome new people!

## Next Month

I will copy last months goals here as I didn't get around to doing a range of damage. I have a few small tweaks I'd like to make, such as mass adding spawners. For a big goal I think adding in a general item listing to the public side of the app would be good. I think it makes sense to add it in because you can view the stats of any item in World of Warcraft, so letting people look around for cool things to hunt out seems like a good addition.

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[elixirlang]: https://twitter.com/elixirlang
