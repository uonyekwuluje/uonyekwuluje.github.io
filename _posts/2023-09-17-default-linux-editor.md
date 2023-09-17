---
layout: post
title:  "Default Linux Editor"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
---

When you complete a linux installation, you will observe that you have editor defaults based on the distribution. You can change this default using different methods. This post is geared at shedding some light on them. 

### Using `select-editor`
To use `select-editor`, type that command on your terminal. You should see the below:
```
Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /usr/bin/code

Choose 1-4 [1]: 
```
Type the number you want. In my case, `vim`.

### Update Environment Variables
You can also change the environment variable to reflect what you want
```
export VISUAL=vim
export EDITOR="$VISUAL"
```
You can use the relative or absolute path.
