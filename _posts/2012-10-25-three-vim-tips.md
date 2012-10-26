---
layout: post
categories:
- vim
title: Three Vim Tips
---

As a ruby programmer, my editor of choice is vim. Here are some cool tips I used earlier this week.

### Macros are in registers

One of the cool things about macros is that when you record a macro you assign it to a register, eg `qq` starts a recording a macro to the `q` register. Once you finish recording the macro you can then paste the macro out and edit it, eg `"qp` will paste the `q` register and `"qy` will yank the selected text into the `q` register. A handy trick to have when you messed up one thing in a macro.

### Recursive Macros

During the SLS internal conference ([slides here](http://www.slideshare.net/smartlogic)), [Paul](https://twitter.com/PaulOstazeski) gave a presentation on vim tips. One of them was a recurisve macro. Using the tip above you can append `@q` to the end of the macro and make it be recursive.

I used this tip to delete a column out of a csv file.

#### WARNING!

Make sure you have the macro go down a line at the end of it, otherwise vim won't hit the end of the file and stop the macro.

### An empty search will use the previous item

When testing out a search I was going to use to mass replace text in vim I used `/` first. It ended up being a pretty complex regex and didn't want to copy and paste from my terminal, surely vim could do better than that. Turns out that it can. If you search for something complex via `/`, when you do a search and replace you can just leave the search section empty and vim will use the complex search you already have. Eg: `%s//new text/`.

This tip was also courtesy of Paul.
