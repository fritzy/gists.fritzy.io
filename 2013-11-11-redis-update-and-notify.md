---
date: 2013-11-11 20:12:00 PST
title: Redis Lua: Updating a Key and Publishing a Logged Notification
tags: redis, lua
type: text/html
---
One of the cool things about Redis Lua scripting is the ability to publish notifications of your changes atomically with those changes made.
There's actually a new Redis feature that can do this for you if configured: [keyspace notifications](http://redis.io/topics/notifications).
But maybe you want to send out a single notification from a command that does several things, and maybe you want to keep a log of those notifications.

<script src="https://gist.github.com/fritzy/7426219.js"></script>

There are other approaches to logging that might be better for your use case.
For example, using an iterator for notification ids, and then knowing that you missed a notification based on the id.
There are a lot of ways of doing this.

P.S. I've been busy today, but I'd like these to be happening every work day, regardless of whether I go to work or not, so here's a late-day entry.
