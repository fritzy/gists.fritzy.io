---
date: 2013-11-08 12:00:00 PST
title: Redis Lua: Collate Redis Key/Value Results
tags: redis, lua
type: text/html
---

Any Redis command that returns key/value pairs returns the results as a single list of alternating keys and values.
If you need to work the results in a Lua script, you'll likely want to collate them in a Lua table.

This script takes a Hash key and returns a collated JSON object of key/values.

<script src="https://gist.github.com/fritzy/7376653.js"></script>


    > HMSET some-hash greeting1 hi
    > HMSET some-hash greeting2 hello
    > HMSET some-hash signoff1 bye

    > EVAL "this script" 1 some-hash
    {"greeting1": "hi", "greeting2": "hello", "signoff1": "bye"}


Keep in mind that there are actually a lot of Redis commands that return key/values this way, not just HGETALL, and that I'm returning JSON just for the eexample's sake.
