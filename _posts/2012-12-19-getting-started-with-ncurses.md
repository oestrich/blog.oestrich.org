---
layout: post
categories:
- ncurses
- ruby
title: Getting Started With Ncurses and Ruby
---

I have been writing an ncurses campfire client [hobostove](http://github.com/smartlogic/hobostove) and recently gave a presentation on using [Ncurses and Ruby](http://www.slideshare.net/SmartLogic/ncurses-in-your-hobostove).

Hobostove uses the [ncurses-ruby](https://github.com/eclubb/ncurses-ruby) gem. It's pretty old but so is Ncurses. Here I'll give the basics of setting up, refreshing, and tearing down. I'll do a panel usage post later.

##### Initialization
    Ncurses.initscr
    Ncurses.cbreak
    Ncurses.noecho

- `Ncurses.initscr` - sets up Ncurses
- `Ncurses.cbreak` - don't wait for a new line to send characters to `Ncurses.getch`
- `Ncurses.noecho` - don't echo typed characters to the screen

##### Update
    Ncurses.doupdate
    Ncurses.refresh

- `Ncurses.doupdate` - compares the virtual screen to the physical screen and updates accordingly
- `Ncurses.refresh` - writes to the terminal

##### Tear down
    Ncurses.echo
    Ncurses.endwin

- `Ncurses.echo` - echos characters back to the screen
- `Ncurses.endwin` - stops ncurses

## Resources
1. [TLDP Ncurses How To](http://tldp.org/HOWTO/NCURSES-Programming-HOWTO/)
