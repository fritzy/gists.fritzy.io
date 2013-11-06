---
date: 2013-11-06 10:00:00 PST
title: Redis Lua: Store JSON as MSGPack and Retrive MSGPack as JSON
tags: redis, lua
---

JSON is nearly universal in the web world, and Redis Lua scripting supports encoding and decoding it, which can be very useful for having advanced logic based on JSON values without having to fully normalize your database.
Redis Lua scripting also has bindings to [MessagePack](http://msgpack.org/), which is a structurally compatible format that is much, much more condensed.
Since we have bindings to both, we can potentially save space, but keep the familiar JSON payload on the client side; why not have the best of both worlds?

<script src="https://gist.github.com/fritzy/7340641.js"></script>

    > EVAL "[the set script]" 1 testkey '{"hello": true}'
    OK

    > GET testkey
    "\x81\xa5hello\xc3"

    > EVAL "[the get script]" 1 testkey
    '{"hello":true}'
