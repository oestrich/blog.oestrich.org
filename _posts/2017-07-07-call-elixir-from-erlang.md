---
layout: post
categories:
- elixir
- erlang
title: Call Elixir from Erlang
description: See how simple it is to call Elixir from Erlang.
date: 2017-07-07 2:30PM
---

I've been reading the [Designing for Scalability with Erlang/OTP][dfseo] book and because of that I've been getting interested in writing some erlang code. I got curious if I could call my elixir code from erlang.

This is very simple. Since module and function names are just atoms in erlang you can call them like this:

```erlang
'Elixir.IO':'puts'("Hello, world!").
```

Elixir puts all of it's modules "under" the `Elixir` atom. So to call an Elixir function named `Api.User.changeset()` you would use `'Elixir.Api.User':'changeset'()`.

[dfseo]: http://shop.oreilly.com/product/0636920024149.do
