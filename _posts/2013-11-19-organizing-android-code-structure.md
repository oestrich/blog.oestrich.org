---
layout: post
categories:
- java
- android
title: Organizing Your Android Code Structure
---

_This post was originally published on the_
[SmartLogic Blog](http://blog.smartlogicsolutions.com/2013/07/09/organizing-your-android-development-code-structure/).

<img class="float-img" alt="A code structure for your android development projects" src="/images/android.jpg" width="225" height="300" />

Writing a medium to large Android app requires having code structure. In creating our latest Android development project, I came across a structure that has helped me out.</p>

## Java Code

I like to have separate files for each class in my Android projects, the only exception being AsyncTasks. Having this many java files means you have to have more packages than the base package. I ended up with a package for each type of main class. Each class is named ending with its type.

* com.example
  * activities
    * Contains all the activities. Classes are all named with Activity at the end. That way, you can immediately know what it is when reading Java code that doesn't have its full package name.
  * adapters
    * Contains all the adapters
  * authenticator
    * Contains any class related to signing a user in. I create a local account and having all related classes together is very handy.
  * data
    * Contains all classes related to data management such as ContentProvider and SQLiteHelper.</p>
  * data.migrations
    * Contains all of my SQLite migrations. I created a class for migrations, [read about it here](http://blog.oestrich.org/2013/02/android-sqlite-migrations/), and put them all in this package.
  * fragments
    * Contains all fragments.
  * helpers
    * Contains helper classes. A helper class is a place to put code that is used in more than one place. I have a DateHelper for instance. Most of the methods are static.
  * interfaces
    * Contains all interfaces.
  * models
    * Contains all local models. When syncing from an HTTP API I parse the JSON into these Java objects using [Jackson](http://wiki.fasterxml.com/JacksonHome). I also pull Cursor rows into these models as well.
  * preferences
    * Contains all classes for custom preferences. When creating the preferences I required a custom PreferenceDialog as well as a custom PreferenceCategory. They live here.
  * sync
    * Contains all classes related to syncing. I use a SyncAdapter to pull data from an HTTP API. In addition to the SyncAdapter a SyncService is required, so I created a package.

## Layouts

The layouts folder can easily become disorganized since you can’t have any folders in it. This is [a known issue](https://code.google.com/p/android/issues/detail?id=2018) and has been ignored for 4 years now. To get around this I name my layout files with a prefix depending on what they are for. This sorts them together in the Eclipse file listing.

* R.layout
  * activity_
  * adapter_
  * fragment_

## IDs
All of my IDs are snake_case. I had a project that was inconsistent with how IDs were named and it was a pain. Many projects seem to do mixedCase notation for IDs, but it seemed weird having the different styles in Java code depending on what type of resource it was, e.g. R.layout.snake_case vs R.id.mixedCase.

## Values

For values I have a separate file for each type of resource, e.g. dimens.xml. This works out well for most of the values, except for strings. There are a large amount of strings in this app and having them all in the same file is pretty unusable. In this case I split out some of the strings into groups by their activity. This allows you to easily find a string if you’re looking at the activity already.

How do you structure your code for Android development projects? Comment and let me know.

For more on Android development, plus Rails and iOS, follow [@SmartLogic](http://twitter.com/smartlogic) on Twitter.

_[Image Source](http://www.fotopedia.com/items/picasaweb-5469559909190709010)_
