---
date: 2013-11-13 14:00:00 PST
title: Redis Lua: Updating a Key and Lookup Hash
tags: redis, lua
type: text/html
---
[Yesterday](http://gists.fritzy.io/2013-11-12-redis-generate-lookup-scan) I mentioned that you should maintain a lookup hash as you update keys.

So today, I decided to put that together.
The only real gotcha is deleting the lookup to the old attribute value before updating the lookup hash.

<script src="https://gist.github.com/fritzy/7458225.js"></script>

Largely the same caviates apply as yesterday.
We could flesh this out to handle many attributes and attributes deeper than the root.
Again, this assumes that each attribute will only exist in one key per lookup hash.
