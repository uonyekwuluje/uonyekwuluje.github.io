---
layout: post
title:  "Maven & Java Development"
categories: Maven
---

This is a collection of maven commands, when and how they are used. I found myself searching and doing the same thing many
times over so I decided to put this down as a quick reference for myself. This assumes you have maven installed and your
java environment configured.

# **Create Package & Exclude Tests**
I find this handy when I have to install or package without testing. This is important if I have a parametised profile
without intended parameters in place.
```
mvn package -Dmaven.test.skip
```
