---
layout: post
categories:
- java
- jdk
title: Top Three Features of Modern Java
description: My top three features of modern Java (7 & 8)
date: 2015-08-18 10:00 am
---

At work we've been doing a project that finally lets me use something past Java 6 (I'm looking at you Android.) I've been really loving it, it's great to be able to use static features with significantly less cruft due to new features of the language.

## Streams And Lambdas

Yesterday I started using streams and I really dig it. They feel very similar to ruby's `Enumerable` methods. Lambdas get pulled into this because if you use streams you kinda have to use lambdas to get the full use of them.

``` java
List<Character> characters
List<String> names = characters.stream()
  .map(character -> character.getName())
  .collection(Collectors.toList());
```

This may still seem longer than doing it in ruby, but compared to what it used to be in Java 6 this is great. We're also still in a fairly verbose language so we can't cut down everything.

## Switch Statements with Strings

This doesn't seem like that great of a thing, and may not be the best way to use a switch statement, but trust me it is. After coming from ruby where you can put anything in a case state, needing to use a big blob of if statements if you had a string was super annoying. 

``` java
switch (value) {
  case "hello":
    System.out.println("Hello");
    break;
  case "world":
    System.out.println(", world!");
    break;
}
```

## IntelliJ

This one is sort of cheating because you can use IntelliJ with Android, but I've started using more of its features recently and I've really started liking it. I've been using Junit 4 with this project and the integration with that is really nice. Gradle is super easy and is pretty similar to bundler and rake mushed together.

## Conclusion

If you haven't tried out Java in a while, I definitely suggest a look back. Go download the latest [IntelliJ][intellij] and [OpenJDK 8][openjdk]. I am going to look into using [Dropwizard][dropwizard] for a new project sometime soon.

[intellij]: https://www.jetbrains.com/idea/
[openjdk]: http://openjdk.java.net/
[dropwizard]: http://www.dropwizard.io/
