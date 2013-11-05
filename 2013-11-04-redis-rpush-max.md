---
date: 2013-11-04 12:00:00
title: Appending to a List and Trimming to Max Size
tags: redis lua
---

I've decided to start publishing a pattern or short snippet of code every day.
Ideally this will help get my creative juices flowing, add some value to my name, and give me material to publish in various forms.

Redis's containers are nice, but the commands are a bit limited.
Lua scripting solves this, allowing us to do atomic business logic on the server itself.

Here's a short script that is pushes a new item to the end of a list, deleting as many left-most entries in order to enforce a maximum size.

<script src="https://gist.github.com/fritzy/7310712.js?file=pushmaxlist.lua"></script>

Example:

    EVAL "script here" 1 mylist "hello1" 3
    EVAL "script here" 1 mylist "hello2" 3
    EVAL "script here" 1 mylist "hello3" 3
    EVAL "script here" 1 mylist "hello4" 3


Now when you look at the contents:

    redis 127.0.0.1:6379> lrange mylist 0 -1
    1) "hello2"
    2) "hello3"
    3) "hello4"
