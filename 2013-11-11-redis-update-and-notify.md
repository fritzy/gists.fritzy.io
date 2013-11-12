---
date: 2013-11-11 12:00:00 PST
title: Redis Lua: Updating a Key and Publishing a Logged Notification
tags: redis, lua
type: text/html
---
One of the cool things about Redis Lua scripting is the ability to publish notifications of your changes atomically with those changes made.
There's actually a new Redis feature that can do this for you if configured: [keyspace notifications](http://redis.io/topics/notifications).
But maybe you want to send out a single notification from a script that does several things, and this keeps a log of notifications.

<script src="https://gist.github.com/fritzy/7426219.js"></script>

You could also use an iterator id so that clients can make sure they've received every notification.

P.S. I've been busy today, but I'd like these to be happening every work day, regardless of whether I go to work or not, so here's a late-day entry.
