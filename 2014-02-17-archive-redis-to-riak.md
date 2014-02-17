---
date: 2014-02-14 12:00:00 PST
title: Archive Redis to Riak
tags: node.js, redis, riak
type: text/html
---

Redis is nice for dealing with data as it is relevant and changing, but once it goes a little stale, you probably want to archive it to free up resources.
Wouldn't it be nice to be able to archive it and still be able to query it?

<script src="https://gist.github.com/fritzy/9057367"></script>

Using my new [redisscan module](https://github.com/fritzy/redisscan), this script scrapes Redis, value by value, diving into the structures, and preserves it in Riak in a normalized way.
You could then query the same keys in Riak or restore it back to Redis at any time.

When I've added advanced pattern matching and Riak indexing, I'll turn this into a standalone project.
For now this is just a proof-of-concept to share.

