---
layout: post
title:  "Vim/vi Tips and Ticks"
categories: Vim
---
Systems administration in linux involves working with files. Lots of files. You have various tools at your disposal, vim, emacs etc.
This post is geared at some of the commands seldom used by me but which makes my config updates cleaner and eases pain of manual changes
at scale


**Removing Whitespaces at the end of a line**<br>
This comes in handy when you need to strip whitespaces at the end of the line. Makes your git commits clean
```
:%s/\s\+$//e
```
