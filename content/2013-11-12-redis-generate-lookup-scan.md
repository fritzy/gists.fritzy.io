---
date: 2013-11-12 11:00:00 PST
title: Redis: Generating a Hash Lookup with SCAN
tags: redis, lua
type: text/html
---

Using a keystore efficiently is all about anticipating how the data will be looked up.
If you can't do that, you may be better using a relational database.

Generating lookup tables and indexes can be helpful for finding the key you're looking for.
For example, if you store users as user:id, but you'd also like to be able to look them up by a username, you can generate a lookup table.

The [SCAN command](http://redis.io/commands/scan) allows us to iterate through keys without getting them all at once and really slowing down the server.
It does this by breaking up the hash of keys into equal spaces and iterating them.
Unfortunately, this approach will give the keys in different iterated results on different servers, because the hash is seeded randomly, making SCAN a non-deterministic command.

This example will fail, only because HSET (any writing command) isn't allowed after using a non-deterministic command (in this case, SCAN).

<script src="https://gist.github.com/fritzy/7436527.js?file=generatelookup.lua"></script>

    [Error: ERR Error running script (call to f_a707d1274859064311a626bf76f47b58f3764c62): @user_script:12: @user_script: 12: Write commands not allowed after non deterministic commands ]

This means that we'll have to do this from the client.
Here is an example of doing this in Node.

<script src="https://gist.github.com/fritzy/7436527.js?file=generatelookup.js"></script>

Basically, in both cases, we call `SCAN [iter]` starting with zero, until iter is 0 again.
For each key, we take the designated attribute, and we set that as the lookup key, and the scanned key as the lookup value.
Now we can use the lookup hash to find the key associated with that attribute.

A few notes:

* This does not handle cases where multiple keys might have the same attribute.
* Using SCAN in this way from the client means that we may miss keys that have been added while scanning. A subsequent update will have to find them.
* Ideally you'll maintain the lookup hash when you set keys, but we'll do that gist another time.
