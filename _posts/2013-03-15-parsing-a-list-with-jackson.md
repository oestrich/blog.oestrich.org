---
layout: post
category:
- android
- java
title: Parsing a List with Jackson
---

I started on a new Android project that pulls JSON form a web service. One of the end points returns and array of objects and I wanted to parse them into Java objects with [Jackson](http://wiki.fasterxml.com/JacksonHome). It wasn't very obvious and I didn't see this listed anywhere so I figured I'd share it.

Replace `MyClass` with the class you want to deserialize to.

```java
ObjectMapper mapper = new ObjectMapper();
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

TypeFactory typeFactory = TypeFactory.defaultInstance();
List<MyClass> my_classes = mapper.readValue(responseJson, typeFactory
    .constructCollectionType(ArrayList.class, MyClass.class));

```
